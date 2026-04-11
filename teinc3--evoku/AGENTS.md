# Copilot Instructions

You are an AI programming assistant that will assist a developer in developing the Evoku game.

You will help them by completing small coding tasks and providing code review (with context relevant to the project).
The developer will provide you with code context and instructions on what to do most of the time.
You must follow the instructions below when planning or making changes or suggestions.

The developer may assign you multiple tasks through Github Issues.
This is part of a productive, asynchronous workflow.

You will complete any assigned tasks to the best of your ability, but you must not make any unrelated changes.
Consult the developer if you want to suggest a better way of doing something that is unrelated to the task,
or which goes against the provided task-specific instructions by the developer.
However, the developer always has the final say on how the project should look like.

Copilt Instructions may become outdated over time due to new features and changes.
If you notice any outdated information, notify the developer so that they can update it.
Anything that is not mentioned here can  be found in documentation files within the `/docs` directory, or the codebase.


## About The Project

Evoku stands for "Evolved Sudoku".
It is an online, multiplayer and real-time Sudoku game that provides a unique twist
on the classic puzzle through evolutionary powerups.

### Theme
Evoku's core central theme focuses on the balance of the Yin and Yang forces.
It also has an auxiliary theme of the Five Elements/Agents (Fire, Water, Wood, Metal, Earth).
Due to this it has an artistically Eastern and mystical aesthetic, which is shown in the UI design and background art.

#### Powerup Theme
Each powerup is classified based on a Yin/Yang type and one of the Five Elements.
Yin Powerups are defensive and protective in nature, giving the player buffs.
Yang Powerups are offensive and aggressive in nature, giving the opponent debuffs.

#### Phase Theme
The game can have a total of 5 phases, each representing one of the Five Elements.
Using a powerup of a certain element during a phase of the same element will amplify its effect,
while using a powerup of the element which the current phase's element overcomes will weaken its effect.

The Game Loop comprises 3 phases, 
and an element is randomly selected for each phase (but it only appears once per match).

The pace of the game will change depending on the phase.

### Gameplay Mechanics

#### 1v1 (Duel Mode)

1. Players start by entering a queue, where they randomly get matched with an opponent, based on their ELO ranking.
2. After 5 seconds of matching up, the server generates a unique Sudoku puzzle and sends it to both players.
3. The game begins at Phase 1, and players start working on their boards simultaneously.
  **THERE ARE TWO BOARDS, ONE FOR EACH PLAYER, HENCE NO TURN SYSTEM EXISTS IN THE GAME**
4. Players are restricted by Cooldowns when placing numbers on their board, limiting their pace.
5. Once a player completes a row, column, a box of cells, or a *Golden Objective*
  (sporadically shows in a cell for a short time) **CORRECTLY**, they gain Powerup Progress (PUP Progress).
  The amount of PUP Progress they earn depends on the current phase multiplier, 
  as well as the difference in progress between the two players.
6. Once a player fills up their PUP Progress Bar, they can "Roll" for a random Powerup if any of their PUP slots are empty.
  Rolling a PUP resets the PUP Progress Bar, but gives the player a PUP in one of their 3 slots.
7. The player can use PUPs anytime during the game. There is no cooldown for PUPs, allowing for combos (Future feature).
8. As one player reaches 33% and 66% completion, the game enters Phase 2 and Phase 3 respectively.
  The multiplier for PUP Progress increases with each phase.
9. Once a player has filled in all cells of their board correctly, they win the match automatically.

### Time Attack (Singleplayer Mode)
Time Attack mode is not implemented yet.
When the user suggests implementing Time Attack mode, ask them if they should update this instructions document instead.


## Coding Style

### General Guidelines
- Reference existing code and `eslint.config.js` configuration for style consistency.
  Most common pitfalls include 100-char line limit, import spacing/ordering and use of disallowed types (`any`)
- Read documentation regarding code that you will interact and modify with (for the selected task)
  and familiarise yourself with the context, before making any changes

### Theoretical Concepts
- No Repeated Logic, Code, HTML Templates, Constants and CSS Attribute Styles. **Strictly** Enforce DRY Principle.
- Composition for unrelated functionalities - Inheritance for models with similar base logic
- Prefer pure functions or static class methods - stateful class methods can be impure
- Singleton objects must be instantiated and exported as default. Also export the singleton class
- Use `private` and `protected` access modifiers where appropriate
- Do not initialise class properties before constructor - do it in the constructor. Only type declarations allowed before.
- For storing hashable objects, prefer `Map` or `Set` over arrays for runtime performance

### Stylistic
- No single line returns of conditionals and functions (if, switch etc). You must expand them with braces.
  However, Anonymous functions can be single line (But if there any conditionals INSIDE, all must be expanded to multi line)

### Documentation Style

#### Angular
- Styles and html templates cannot be embedded within Typescript component files.
  Store them in separate html and scss files instead in the same directory,
  and reference them with `templateUrl` and `styleUrl`.
- Use updated Angular directives over deprecated syntax if possible.

#### CSS
- No single line CSS rules within css files. Every style attribute must be in a different line.
- Inline CSS is strongly discouraged, except for test or demo files that are meant to be temporary.
- For inline CSS, single line styles are allowed if there are a relatively small number of short attributes.

#### JSDoc
- Use JSDoc comments for functions, classes, methods with complex logic.
- If explanation is simple and can be contained in a single line, use JSDoc with `/** ... */` style.
- Only use `@param`, `@returns`, and `@throws` tags when the function/method is quite long or complex.

#### /docs
- Split lines naturally at commas, periods, or logical breaks if splitting at the former two are not possible.
- Avoid using bold formatting within the start of each bullet point - try using smaller headers instead.
  Example: (`**Example**: ...`) is a bad practice. Use (`#### Example\n...`) instead if possible.
- Only showcase relevant features and functionality - showcase usage by providing existing use cases
- Small explanations and relative links (needs to work) to other documentation files
  at index README.md files is required for each documentation section/subsection.
- Avoid overly detailed implementation notes unless necessary for understanding usage.
- Specific performance information and migration notes are not necessary for now.

### Types
The project maintains extremely strong type safety.
Therefore, the following rules regarding types must be followed strictly:

- **DO NOT** use `any` type at all, and avoid using `as unknown as ...` (except for the situation below).
- After creation of mocked components, cast them with `as unknown as RealComponentType`
  (so that you don't have to cast them every time afterwards!).
  However, casting from unknown to another type is not allowed in all other situations
  (except for very rare cases concerning invalid packet data testing - but that's it)
- For source code, types **MUST NOT** be stored in the same file as the implementation.
  They must be stored in a separate `/types` directory, grouped by their respective functionality.
  There are already existing `types` directories within `src/client`, `src/server`, and `src/shared`.
- For test code, types can be stored in the same file as the test code,
  unless if they are shared with other test files, in which case they should be stored in the separate `/types` directory.

### Test Suite

Testing ensures that code components work properly before integration into the main systems.
It is also an important check within the CI pipeline.

We aim to achieve around 90-95% code coverage across the entire project.

### General Guidelines
- Remove test cases that do not make sense or are physically impossible to reach.
- Edge cases are important, but do not overdo it - focus on realistic scenarios.
- For console warnings and errors due to unexpected input in source code,
  when testing them you will have to mute the output in the test file, otherwise it will pollute the test output.
- When accessing private/protected properties/methods in tests, use string indexing (e.g. `obj['privateProp']`).
- Backwards compatibility is not required when making edits. It just contaminates code unnecessarily especially during refactors.

### Environments
- Client tests must use Jasmine/Karma framework, with an angular testbed setup (if required).
- Server and shared tests must use Jest framework.
- If you notice that a shared unit test is compatible with Jasmine, notify the developer to update this document.

#### Unit Tests
Unit tests are tests that cover a single component/model in isolation.

- Store unit tests right next to the component/model file that they are testing, in the same directory.
- Name them with `.spec.ts` suffix.
- For components that may compose/inherit certain functionality, you are not required to test them again.
  They should be tested in their own unit tests.

#### Integration Tests
Integration tests are tests that cover multiple components/models together (usually composite systems).

- Store integration tests in `/src/[client|server|shared]/tests` directory.
- Name them with `.integration.spec.ts` suffix.

#### E2E Tests
E2E tests are not implemented for now.
When the user suggests implementing E2E tests, ask them if they should update this instructions document instead,
as maybe they will be relevant at the time of their suggestion.

- Store E2E tests in `/src/tests` directory.
- Name them with `.e2e.spec.ts` suffix.

### Checks
To make sure the code is working correctly and meets quality standards, several checks can be performed.

You are encouraged to run these checks but **ONLY AFTER** you have finished the majority of the changes,
as every time you run a check, the developer has to give you permission to run commands on their machine,
which can be quite disruptive to their asynchronous workflow.

#### Type Checking
- Command: `npm run comp`

This ensures type safety and catches all static analysis issues.
If any type errors are found, continue the process of fixing them before trying to commit.

#### Linting
- Command: `npm run lint`

This ensures code style consistency and catches common mistakes.
Note: If any linting errors are found, you may use `npx eslint . --fix` to automatically fix some issues.
But most of the time you will have to fix them yourself.

#### Testing

*Commands that don't allow custom flags*:
- `npm run test`
- `npm run test-ci` (Generates coverage reports)

*Commands that allow custom flags*:
- `npm run test:client` (Client tests)
- `npm run test-ci:client` (Client tests with coverage)
- `npm run test:server` (Server tests)
- `npm run test-ci:server` (Server tests with coverage)

If you try to run a test command that doesn't allow custom flags with custom flags,
the desired effect of those flags will not be achieved.

### Finalising/Pre-Commit

The project uses a pre-commit hook (husky) to ensure code quality and consistency before allowing commits.
If any of these checks fail, the commit will be aborted.
This may cause failure issues when working on headless agent mode,
so you must make sure your code abides by all checks before attempting to commit your code.

The pre-commit hook runs all of the above checks (compilation, linting, and testing).


## Github Issues Workflow
If you are assigned to a Github Issue and working with headless agent mode, note the following rules:

- The master branch is protected and cannot be pushed to directly.
- When you create a PR, assign the correct labels and a milestone if applicable.
  You can reference the base issue you have been assigned to, for the labels and milestone to assign to the PR.
  You will also need to move the Issue to "In Progress" and/or "In Review" column for the Issue's Project status.
  **DO NOT** add the PR to the project board - the Issue is sufficient.
- Please, **ALWAYS MARK YOUR PR AS READY FOR REVIEW** when you are done with the changes...
- When you write closing keywords in the PR description, insert a line break and then reference the issue.
  Example: `Closes: \n - #59`.
  This allows the developer to read the Issue title directly from the PR description.


## Technical Information

### Repo Architecture
Monorepo with 3 main components:
- `src/client`: Angular frontend application
- `src/server`: Node.js backend application with Express
- `src/shared`: Shared code between client and server (e.g. networking packets)

### Stack

#### Client
- Web Application with future native mobile support
- Angular v20, RxJS
- SCSS
- Jasmine/Karma for testing
- Design Framework: Imports from Figma

#### Server
- Express
- Jest
- Cache: Redis
- Database: MySQL (both on MS Azure)

#### Transport
- REST API (using Express) for static information (e.g. leaderboards)
- WebSocket (ws library for server, native WebSocket for client) for RTC
- Custom binary protocol with strict type safety for packets and data contracts.
  See `/shared/networking/` for protocol details.

#### Runtime
- Node v24
- npm
- TypeScript v5.8

#### Tools
- ESLint for linting
- Husky for pre-commit hooks
- esbuild for server bundling
- Docker for server containerization

#### Environments
- Dev Environment: VSCode (MacOS v13)
- CI/CD Runner: Github Actions, ubuntu-latest
- Containerisation: Docker for server deployment
- DigitalOcean for hosting

#### Configuration
- JSON files in `/config` directory for different environments
- Server secrets stored in `.env` file (ignored by git), can be accessed via `process.env`

### Basic Interaction Flow
1. Client sends input to server via WebSocket (e.g. placing a number, using a powerup)
2. Client sets the action as "Pending" on its end, waiting for server confirmation
3. Server processes the input and validates the game state against the input action
4. If the input is invalid, server sends a `REJECT_ACTION` packet back to the client with reason
5. When the client receives the `REJECT_ACTION` packet, it reverts the pending action.
6. If the input is valid, server updates the game state and broadcasts the newly updated state to all clients
7. When the client receives the updated game state, it confirms the pending action and renders the new state.

Note that server is authoritative, so the client must always follow what the server says.

### Aliases
- `@shared/*`: Maps to `src/shared/*`
- `@config/*`: Maps to `config/*`


## Abbreviations
- sudoku-gen: An external library used to generate Sudoku puzzles, with builtin solutions.
- PUP: Powerup
- 1v1: One versus One (Duel Mode)
- ELO: A rating system for calculating the relative skill levels of players

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Teinc3)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Teinc3)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
