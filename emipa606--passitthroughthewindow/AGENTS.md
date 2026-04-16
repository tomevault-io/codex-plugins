# GitHub Copilot Instructions for "Pass It Through The Window! (Continued)"

## Mod Overview and Purpose
"Pass It Through The Window! (Continued)" is an update to the original mod by SilkCircuits, which introduces a two-sided pass-through window to facilitate the transfer of items between adjacent rooms without disrupting the climate of either room. This mod is tailored for innovative, climate-controlled logistics, making it an excellent tool for unique scenarios, from enhancing kitchen operations to stealthy deliveries, encouraging efficient and unobtrusive item transfer methods.

## Key Features and Systems
- **Climate-Controlled Transfer**: Enables item transfer between adjacent rooms with no temperature loss.
- **Automatic Alignment**: Adjusts to the correct wall orientation (horizontal or vertical) automatically.
- **Storage Capacity**: Can hold up to 3 items stacks, but is unsuitable for chunks, plants, or buildings.
- **Construction Requirements**: Requires 20 steel, 1 component, and 20 units of any material (e.g., wood, stone).
- **Research Dependency**: Unlocks after researching Complex Furniture.
- **Compatibility and Categorization**: Appears under the Furniture tab (or Storage tab if using LWM’s Deep Storage).

## Coding Patterns and Conventions
- **Namespace and Class Structures**: Follow the conventions seen in the `.NETFramework,Version=v4.8.AssemblyAttributes.cs`, ensuring that classes are well-organized and meaningful.
- **OOP Principles**: Utilize object-oriented programming effectively; classes like `Building_PassThroughWindow` should inherit from appropriate base classes like `Building_Storage`.
- **Code Readability**: Maintain clear and descriptive class and method names, alongside comprehensive comments to ensure the code is easily understandable and maintainable.

## XML Integration
- **Defining New Buildings**: Make use of XML files to define and configure the new pass-through window building, detailing its resource requirements, tech dependencies (Complex Furniture), and placement parameters.
- **XML Modifications**: Ensure XML entries are robust to integrate with RimWorld's existing systems, allowing adaptability with mods like LWM's Deep Storage.

## Harmony Patching
- **Patch Implementation**: Utilize Harmony for non-destructive patching if modifications to the game engine or interactions with specific methods are required.
- **Conflict Minimization**: Identify potential methods that may require patching and ensure compatibility across different game versions or other mods by properly annotating patches.

## Suggestions for Copilot
1. **Contextual Understanding**: Assist in generating and adapting classes relevant to item storage and transfer within the RimWorld context, taking into consideration the item's climate-controlled premises.
2. **XML Handling**: Provide guidance in structuring XML definitions, ensuring consistency with RimWorld's architecture for introducing new buildables.
3. **Harmony Patching Advice**: Leverage Harmony for seamless integration; suggest points of injection for methods where game logic conflict might occur.
4. **Efficiency and Optimization**: Offer optimized solution suggestions for item transfer logic, tapping into efficient execution while maintaining gameplay balance.
5. **Comprehensive Documentation**: Encourage writing documentation for public methods, indicating their purpose, parameters, and return types, enhancing codebase understandability for future developers.

By adhering to these guidelines and leveraging GitHub Copilot's capabilities, developers can expedite the process of updating and enhancing the "Pass It Through The Window!" mod, ensuring a seamless and coherent integration into the RimWorld gameplay experience.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emipa606) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
