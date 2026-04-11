# GitHub Copilot Instructions for FoundryVTT RPG Spell Animation Project

## Project Overview

This is a **FoundryVTT spell animation collection** designed specifically for a **custom RPG system** using the Sequencer module. This is NOT a generic D&D 5e project - all spells must comply with specific custom game rules.

### Essential Documentation

- **GAME-RULES.md** - Complete RPG mechanics (REQUIRED reading)
- **DEVELOPMENT.md** - Installation, setup, and development guidelines
- **MODULE-REQUIREMENTS.md** - Module dependencies and compatibility
- **assets/README.md** - Effects and assets reference

### Core Purpose

- Create **visual spell animations only** (no game mechanics automation)
- Support **custom RPG turn-based combat** with specific rules
- Provide **RPG-compliant spell macros** that work with the custom game system
- Maintain **clear separation** between animation/visuals and actual game calculations

**Reference Implementation**: `/macros/characters/ora/bubbles.js` showcases full RPG integration

- **Initiative**: Based on Agilité characteristic
- **Actions per Turn**: 1 Simple Action + 1 Movement Action
- **Movement**: Up to 6 squares, diagonal costs 1.5 (rounded down)

### Combat Stances (Choose at turn start)

1. **Offensive**: Maximize all damage dice, -2 Agilité for defense
2. **Defensive**: Can use reactive spells, use spell stats for defense
3. **Focus**: Most spells become free mana cost, -1 Agilité

### Resources

- **Health Points**: 10 base + bonus from creation points
- **Mana Points**: 5 base + bonus from creation points
- **Critical**: 0 HP or MP = unconscious/danger

### Spell Mechanics

- **Damage Formula**: Varies by spell, often `1d6 + (Stat + bonus)/2`
- **Mana Costs**: Specific per spell, many are "focusable" (free in Focus stance)
- **Spell Levels**: Start at 1, gain +2 hit bonus per level
- **Element Effects**: Create tactical advantages/vulnerabilities

## Technical Stack

### Available Modules (Pre-installed on Server)

1. **Sequencer** (ESSENTIAL) - Core animation system
2. **JB2A - Jules&Ben's Animated Assets (Free)** (ESSENTIAL) - Visual effects library
3. **JB2A - Jules&Ben's Animated Assets (Patreon)** (ESSENTIAL) - Extended visual effects
4. **Portal** (REQUIRED) - Crosshair targeting system
5. **Carousel Combat Track** (REQUIRED) - Turn order management
6. **Animated Spell Effects** (AVAILABLE) - Additional spell effects library
7. **Animated Spell Effects - Cartoon** (AVAILABLE) - Cartoon-style spell effects

**Note**: All required modules are pre-installed on the server. Spells can use any combination of these effect libraries.

### Verified APIs

```javascript
// Combat state access
const combat = game.combat;
const isActive = combat.started;
const activeCombatant = combat.combatant;
const currentRound = combat.round;
const currentTurn = combat.turn;

// Turn validation pattern
function validateSpellCasting(casterToken, spellName) {
  const combat = game.combat;
  if (!combat?.started) {
    ui.notifications.warn(`${spellName} requires active combat`);
    return false;
  }

  const activeCombatant = combat.combatant;
  if (activeCombatant?.token?.id !== casterToken.id) {
    ui.notifications.warn(`${spellName}: Wait for your turn!`);
    return false;
  }

  return true;
}
```

## Coding Standards

### Spell Macro Structure

```javascript
/**
 * Spell Name - Brief Description
 *
 * RPG-COMPLIANT SPELL for Custom RPG System
 *
 * Description: Detailed spell mechanics
 * Damage/Healing: Formula explanation
 * Mana Cost: X mana (focusable/not focusable)
 * Prerequisites: Required modules
 */

(async () => {
  // 1. Validate requirements
  if (!canvas.tokens.controlled.length) {
    ui.notifications.error("Please select your character token first!");
    return;
  }

  // 2. Get user inputs (dialogs for stats, options)

  // 3. Handle targeting (Portal crosshairs)

  // 4. Roll damage/healing for display

  // 5. Create Sequencer animation

  // 6. Display results in chat

  // 7. Helper functions
})();
```

### Animation Patterns

```javascript
// Basic Sequencer pattern
new Sequence()
  .effect()
  .file("jb2a.effect.webm")
  .atLocation(caster)
  .scale(0.8)
  .sound()
  .file("path/to/sound.wav")
  .volume(0.3)
  .play();

// Projectile pattern
new Sequence()
  .effect()
  .file("jb2a.projectile")
  .atLocation(caster)
  .stretchTo(target)
  .waitUntilFinished(-200)
  .effect()
  .file("jb2a.explosion")
  .atLocation(target)
  .play();
```

## File Organization

```
/macros/
├── basic/           # Simple single-effect spells
├── intermediate/    # Multi-step sequences
├── advanced/        # Complex targeting & interactions
├── spells/          # Generic spell animations
├── characters/      # Character-specific RPG-compliant spells
└── utilities/       # Helper functions

/examples/           # Non-compliant examples (for learning only)
└── characters/      # Old character spells (don't copy these)

/templates/          # Spell macro templates
/docs/              # Documentation
```

## What Spells Should Include

### ✅ Required Elements

- **RPG Rule Compliance**: Follow custom game mechanics
- **User Input Dialogs**: For stats, options, targeting
- **Portal Targeting**: For crosshair selection
- **Damage/Healing Display**: Show formula and results in chat
- **Proper Error Handling**: Validate inputs and requirements
- **Clear Documentation**: Comments explaining RPG mechanics

### ✅ Animation Elements

- **Casting Effects**: Visual feedback on caster
- **Projectile/Effect**: Main spell animation
- **Impact/Result**: Visual conclusion
- **Sound Effects**: Audio feedback
- **Timing**: Proper delays and synchronization

### ❌ What NOT to Include

- **Automatic Dice Rolling**: Let external systems handle this
- **Character Sheet Automation**: Don't modify actor data
- **Turn Order Management**: Carousel Combat Track handles this
- **Mana Deduction**: Character sheet manages resources
- **Damage Application**: Show results only, don't apply

## RPG-Specific Guidelines

### Spell Categories by Stat

- **Physique**: Physical-based spells (force, impact)
- **Dextérité**: Precision spells (targeting, fine control)
- **Esprit**: Mental/concentration spells (most magic)
- **Sens**: Perception-based spells (detection, awareness)
- **Volonté**: Will-based spells (resistance, determination)
- **Charisme**: Social/emotional spells (charm, fear)

### Focus Mechanics

- **Focusable Spells**: Become free (0 mana) in Focus stance
- **Non-Focusable**: Always cost mana regardless of stance
- **Half-Focus**: Mana cost reduced by half in Focus stance

### Element Interactions

- **Water**: Often reduces speed/movement
- **Ice**: Creates electrical damage vulnerability
- **Fire**: Direct damage, synergizes with oil
- **Oil**: Creates fire damage vulnerability
- **Lightning**: Benefits from ice vulnerability
- **Shadow**: Life drain, stealth effects
- **Living Water**: Healing variant, often non-focusable

## Chat Message Format

```javascript
const chatContent = `
    <div class="spell-result">
        <h3>🎯 Spell Name (Element Type)</h3>
        <p><strong>Caster:</strong> ${actor.name}</p>
        <p><strong>Targets:</strong> ${targetDescription}</p>
        <p><strong>Mana Cost:</strong> ${manaCost}</p>
        <hr>
        <p><strong>Damage/Healing:</strong> ${result}</p>
        <hr>
        <p><strong>Element Effect:</strong> ${elementEffect}</p>
    </div>
`;
```

## Example: Complete Spell Implementation

See `/macros/characters/ora/bubbles.js` for a full RPG-compliant spell example with:

- Four element variants (Water, Ice, Oil, Living Water)
- Proper focus mechanics
- Self-targeting capability
- Dual/single projectile logic
- Complete user interaction flow

## Important Notes for Copilot

1. **Always ask for stat values** - cannot be auto-retrieved from character sheets
2. **Focus on visual effects** - this is an animation collection, not a game system
3. **Follow RPG rules strictly** - this is NOT generic D&D 5e
4. **Use proper error handling** - validate all user inputs
5. **Include comprehensive documentation** - explain RPG mechanics in comments
6. **Test targeting scenarios** - single target, multi-target, self-target
7. **Consider stance implications** - how spell behaves in different combat stances

## Project Status

- **Documentation**: Complete with verified API capabilities
- **Examples**: Moved to `/examples/` (non-compliant reference only)
- **RPG Compliance**: All new spells must follow custom rules
- **Character Development**: Ora has working RPG-compliant Bubbles spell
- **Future**: Expanding character spell collections with proper RPG integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xaxzie)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/xaxzie)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
