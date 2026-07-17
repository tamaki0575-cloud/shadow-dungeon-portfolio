# Architecture Overview

## Scope

This document describes the system at a conceptual level. Names identify architectural responsibilities and selected domain concepts; they do not reproduce source code, class members, assignment data, or course-provided configuration.

## High-Level Structure

Shadow Dungeon follows a layered real-time update flow:

1. The **Bagel game engine** provides a frame callback and the current input snapshot.
2. The **game controller** identifies the active game state and coordinates top-level transitions.
3. **Room and state management** delegates the frame to the current preparation, battle, end, or game-over context.
4. The active room updates the **player** and its composed **entities**.
5. The logical **collision system** detects overlaps and applies gameplay-specific outcomes.
6. State changes feed back into progression, while the **UI system** renders health, currency, equipment, keys, prompts, and completion messages.

The accompanying diagram deliberately uses responsibility-level labels. For example, Collision System represents collision logic distributed across room and entity behaviour rather than asserting the presence of a single collision-manager class.

## Module Relationships

### Game engine

The framework owns the application window and repeatedly invokes the game update operation. It supplies input and drawing primitives but does not own Shadow Dungeon’s domain rules.

### Game controller

The controller is the coordination boundary for initialisation, the active room, global overlays, restart behaviour, and transitions. It connects the framework lifecycle to the game-specific model.

### Input system

Keyboard state enters through the framework and is passed to the component that owns each action. Movement and firing reach the player system; interaction keys reach items or doors; global commands reach the controller or overlay responsible for them.

### Room manager and state management

Rooms group the data and rules needed for a particular environment. Preparation, combat, and ending contexts have different responsibilities, while battle rooms additionally manage enemy activation, completion checks, locked exits, drops, and rewards.

The implementation uses explicit state-oriented coordination. This keeps the flow visible for a project of this size, although a production version would benefit from a common state interface and transition object.

### Player system

The player model owns position, previous position, health, currency, character choice, weapon progression, keys, facing direction, and action cooldowns. It transforms input into movement or attacks and exposes bounded operations through which other objects apply damage, rewards, purchases, or collectibles.

### Entity system

The entity model is built from several focused families:

- **Enemies** share health, activation, contact damage, rewards, firing cooldown, and lifecycle rules. Variants provide specialised attack or movement behaviour.
- **Projectiles** share position, direction, speed, damage, allegiance, collision state, and draw/update behaviour.
- **Items and interactables** include keys, treasure, breakable containers, hazards, obstacles, doors, restart areas, and store interactions.
- **Environment objects** constrain movement or modify the player while overlap conditions hold.

Rooms compose these objects rather than inheriting from them, reflecting ownership and lifecycle more accurately.

### Collision system

Collidable objects expose a common overlap capability based on axis-aligned bounds. Collision detection answers whether two objects intersect; resolution then applies domain-specific consequences such as:

- reverting blocked movement;
- damaging a player or enemy;
- consuming a projectile;
- breaking an obstacle;
- granting currency;
- collecting or spending a key;
- unlocking or traversing a door;
- removing an object that leaves the window.

Keeping detection consistent while making resolution explicit was practical for the limited number of interactions. As the interaction matrix grows, a dedicated resolver would reduce room complexity.

### UI system

The UI layer renders player statistics, prompts, character descriptions, and end-state messages. It reads current state but should not decide gameplay outcomes.

## Runtime Flow

### Per-frame lifecycle

1. Receive the current input snapshot.
2. Process global commands or overlays.
3. Select the active room/state.
4. Update room-owned environment and interactable objects.
5. Update active enemies and allow them to create projectiles.
6. Update the player and allow player actions to create projectiles.
7. Advance projectiles and resolve relevant collisions.
8. Apply rewards, damage, drops, completion, or transition effects.
9. Render the current room, entities, player, and UI.

### Transition flow

A door or end condition requests a state change. The controller transfers the player to the destination context, clears transient projectiles where necessary, and prevents the previous room from continuing its current update. Encounter completion can unlock exits, while restart logic recreates initial state.

### Combat data flow

Input produces a player attack request. The player system creates a friendly projectile carrying direction and damage data. The active room advances it and checks candidate overlaps. A hit updates the target lifecycle, marks the projectile for removal, and may trigger currency or item-drop effects. Enemy attacks follow the same movement model with hostile allegiance and a player-facing outcome.

## Architectural Boundaries

The strongest boundaries are the reusable enemy/projectile families, the common overlap capability, and room composition. The main opportunities for improvement are reducing global/static coordination, separating update from rendering, consolidating repeated room behaviour, and extracting collision outcomes from large controllers.

