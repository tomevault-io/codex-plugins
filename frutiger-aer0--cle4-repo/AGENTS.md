# Excalibur.js Core Instructions

You are an AI programming assistant helping create JavaScript games with ExcaliburJS. The user is a student learning game development in a team environment.

## Code Style & Conventions

### Naming Conventions
- **Classes**: PascalCase (e.g. `GameManager`, `PlayerController`)
- **Variables/functions**: camelCase (e.g. `currentHealth`, `movePlayer()`)
- **Constants**: UPPER_SNAKE_CASE (e.g. `MAX_HEALTH`)
- **Files**: lowercase only (e.g. `player.js`, `gamemanager.js`)
- **Private members**: Use `#` prefix

### Comments in Dutch
```javascript
// Deze functie wordt één keer aangeroepen wanneer de speler wordt toegevoegd
// Zie het als het "klaarmaken" van de speler voordat het spel begint
onInitialize(engine) {
    this.graphics.use(Resources.Player.toSprite());
}
```

## Core Development Rules

### Build Tool
- Use **Vite** for development and building
- Use Excalibur CLI: `npx @excaliburjs/cli@latest`
- Always use ES6 imports, never `require()`

### Engine Basics
- ExcaliburJS has its own game loop - NO `requestAnimationFrame`
- Use `onInitialize()` for setup, not constructor
- Use `onPostUpdate()` for per-frame logic
- never use setTimeout or setInterval, if you need to track frames, just use a frameCounter / cooldown counter

### Movement

```js
// Movement
this.vel = new Vector(x, y)
```

### Collision Detection
```javascript
hitSomething(event) {
    if(event.other.owner instanceof Shark) {
        console.log("fish hits a shark")
    }
}
```

## Project Structure

```
public/
├── dialogue/
│   └── testdialogue.json
└── images/
src/
├── css/
│   └── style.css
├── js/
│   ├── actors/              # Game characters/entities
│   │   ├── npc.js           # Parent NPC class
│   │   └── testactor.js     # test NPC implementation
│   ├── player/              # Player characters
│   │   ├── robot/           # Robot player type
│   │   │   ├── mouse.js     # Mouse interaction
│   │   │   ├── player.js    # Robot player (nog leeg)
│   │   │   └── pointer.js   # Pointer interaction
│   │   └── shanty/          # Shanty player type
│   │       └── shanty.js    # Shanty character
│   ├── scenes/              # Game scenes
│   │   ├── cutscenes/       # Story scenes
│   │   │   ├── restaurantscene_1.js
│   │   │   ├── restaurantscene_2.js
│   │   │   ├── restaurantscene_3.js
│   │   │   ├── restaurantscene_4.js
│   │   │   └── testscene.js
│   │   ├── locations/       # Reusable locations
│   │   │   ├── restaurant.js
│   │   │   └── worldmap.js  # (nog leeg)
│   │   └── minigames/       # Interactive gameplay
│   │       ├── minigame_1/
│   │       │   ├── background_1.js
│   │       │   ├── minigame_1.js
│   │       │   └── tree.js  # (nog leeg)
│   │       ├── minigame_2/
│   │       │   ├── background_2.js
│   │       │   ├── food.js
│   │       │   └── minigame_2.js
│   │       └── minigame_3/
│   │           ├── background_3.js
│   │           ├── dock.js
│   │           ├── minigame_3.js
│   │           ├── platform.js
│   │           ├── sea.js
│   │           └── turtle.js
│   ├── debug_control.js     # Development debugging
│   ├── game.js             # Main game entry point
│   └── resources.js        # Asset management      
```

## Basic Templates

### Main Game (game.ts)
```javascript
import { Engine, DisplayMode } from "excalibur"
import { Resources, ResourceLoader } from './resources'

export class Game extends Engine {
    constructor() {
        super({
            width: 1280,
            height: 720,
            maxFps: 60,
            pixelRatio:1,
             physics: {
                gravity: new Vector(0, 800),
                solver: SolverStrategy.Realistic
            },
            displayMode: DisplayMode.FitScreen
        })
        this.start(ResourceLoader).then(() => this.startGame())
    }

    startGame() {
        this.add('menu', new MenuScene())
        this.add('game', new GameScene())
        this.goToScene('menu')
    }
}

new Game()
```

### Actor Template
```javascript
import { Actor, Vector, CollisionType } from "excalibur"
import { Resources } from '../resources'

export class Player extends Actor {
    constructor(pos) {
        super({
            pos,
            width: 32,
            height: 32,
            collisionType: CollisionType.Active
        })
    }

    onInitialize(engine) {
        // Setup graphics en collision
        this.graphics.use(Resources.Player.toSprite())
        
        // Event listeners
        this.on("collisionstart", (evt) => this.onCollision(evt))
    }

    onCollision(evt){
        console.log(evt.other.owner)
    }

    onPostUpdate(engine, delta) {
        // Per-frame logic
        this.handleInput(engine)
    }
}
```

### Resources (resources.js)
```javascript
import { ImageSource, Sound, Loader } from 'excalibur'

export const Resources = {
    Player: new ImageSource('images/player.png'),
    Enemy: new ImageSource('images/enemy.png'),
    JumpSound: new Sound('sounds/jump.mp3')
} as const

export const ResourceLoader = new Loader()
for (const res of Object.values(Resources)) {
    ResourceLoader.addResource(res)
}
```

### Scene Template
```javascript
import { Scene, Vector } from 'excalibur'
import { Player } from '../actors/player'

export class GameScene extends Scene {
    onInitialize(engine) {
        // Maak actors en voeg toe aan scene
        const player = new Player(new Vector(100, 300))
        this.add(player)
    }
    
    onActivate() {
        // Wordt aangeroepen als scene actief wordt
        Resources.BackgroundMusic.play({ loop: true })
    }
    
    onDeactivate() {
        // Cleanup bij verlaten scene
        Resources.BackgroundMusic.stop()
    }
}
```

## Essential Best Practices

### Memory Management
- Avoid creating new Vectors in update loops
- Remove event listeners when actors are killed
- Stop sounds when switching scenes

### Collision Types
- `Active`: Moving objects (player, enemies)
- `Fixed`: Immovable objects (walls, floors)
- `Passive`: Triggers (collectibles)
- `PreventCollision`: No collision at all

### Common Pitfalls
1. **Forgetting to add actors to scenes** - They won't render!
2. **Using constructor instead of onInitialize()**
3. **Wrong event names** - Use 'pointerdown' not 'click'
4. **Creating objects in update loops** - Causes GC issues
5. **If resources loading fails, the whole game will fail and give strange errors, double check all loading code**

### Input Handling
```javascript
onPreUpdate(engine) {
  let xspeed = 0;
  let yspeed = 0;
  
  if (engine.input.keyboard.isHeld(Keys.Left)) {
      xspeed = -this.speed;
  }
  
  if (engine.input.keyboard.isHeld(Keys.Right)) {
      xspeed = this.speed;
  }
  
  if (engine.input.keyboard.isHeld(Keys.Up)) {
      yspeed = -this.speed;
  }
  
  if (engine.input.keyboard.isHeld(Keys.Down)) {
      yspeed = this.speed;
  }
  
  this.vel = new Vector(xspeed, yspeed);
}
```
Beweging beperken
Als je wilt voorkomen dat het object het scherm uitgaat, kan je gebruik maken van de functie clamp. Deze zit in het Excalibur framework.
```javascript
this.pos.x = clamp(this.pos.x, this.width / 2, engine.drawWidth - this.width / 2);
this.pos.y = clamp(this.pos.y, this.width / 2, engine.drawHeight - this.height / 2);


## Debugging

- Enable debug mode: `game.showDebug(true)`
- Check browser console for warnings
- Use Chrome Excalibur DevTools extension

Remember: Focus on game logic. Let Excalibur handle the engine details!

## Troubleshooting
Common Issues
Assets not loading

Check pad vanaf public folder
Zorg dat assets in git staan
Clear browser cache

Collision not working

Check CollisionType settings
Verify actor is added to scene
Enable debug mode om hitboxes te zien

Memory leaks

Stop alle sounds in onDeactivate
Remove event listeners
Clear timers/intervals

## Code Review Checklist

 Comments: Zijn functies in het Nederlands gecomment?
 Naming: Volgt code de conventions? (PascalCase, camelCase, etc.)
 Private: Zijn private members gemarkeerd met #?
 Lifecycle: Wordt onInitialize() gebruikt ipv constructor?
 Functions: Geen geneste functies? Kleine, duidelijke functies?
 Cleanup: Worden resources opgeruimd in onDeactivate()?
 Console: Geen console.log statements achtergelaten?
 Types: gebruik alleen javascript files en geen typescript files

---
> Source: [Frutiger-AER0/CLE4-repo](https://github.com/Frutiger-AER0/CLE4-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-23 -->
