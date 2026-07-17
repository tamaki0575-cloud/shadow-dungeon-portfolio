# Design Decisions

## Retrospective Context

Shadow Dungeon was scoped as an object-oriented software design project rather than a general-purpose game engine. The implementation therefore favoured visible domain concepts and straightforward control flow over a large abstraction framework. This retrospective distinguishes useful decisions at that scale from changes that would be appropriate if the game became a maintained production system.

## Object-Oriented Modelling

OOP suited the problem because the game consists of persistent objects with identity, state, and behaviour: players take damage, enemies transition from inactive to active to defeated, doors lock and unlock, projectiles move and expire, and items change collection state.

Keeping these transitions with the responsible object limits the number of components that can mutate its internals. It also makes the model easier to explain in domain terms than a collection of unrelated procedural operations.

## Shared Behaviour Through Inheritance

Enemy variants share a genuine lifecycle: activation, health, contact damage, cooldown tracking, rewards, collision bounds, and death. Projectile variants similarly share movement, damage, allegiance, rendering, and expiry state. Base abstractions were therefore used to centralise invariant behaviour while allowing specialised actions.

The enemy update flow resembles Template Method: common lifecycle steps are fixed, and a variant supplies the action that differs. This improves consistency but also couples each variant to the base lifecycle. The trade-off is acceptable while enemy families remain small and behaviour is primarily type-specific.

For a larger game, some differences—movement, targeting, weapons, and drops—would be better represented as composed strategies. That would allow behaviours to be mixed without creating a subclass for every combination.

## Composition for Rooms and Encounters

Rooms contain doors, obstacles, enemies, projectiles, and items. Composition models that ownership directly and gives each room a clear lifecycle boundary. It also makes encounter completion a room-level rule because the room can inspect the entities it owns.

The trade-off is that room controllers can accumulate too many orchestration responsibilities. The battle-room implementation became the natural home for update ordering, rendering, collision outcomes, drops, rewards, and door unlocking. A production refactor would keep composition while delegating those responsibilities to narrower services.

## Capability Interface for Overlap

Players, enemies, projectiles, doors, obstacles, and items are not all members of one meaningful domain hierarchy, but they all need collision bounds. A small capability interface provides a shared overlap contract without forcing unrelated objects under an artificial base class.

This keeps geometric detection reusable. Gameplay consequences remain explicit because the same overlap can mean damage, blocking, collection, unlocking, or transition depending on the participating objects.

## State-Oriented Room Flow

The game is naturally divided into preparation, encounter, completion, and end phases. Representing these phases with room-focused components keeps state-specific data together and prevents the main loop from directly managing every entity.

The project uses state-oriented organisation rather than a formal State pattern. Central transition logic is simple to trace at this scale, but it introduces conditional branching and global coupling. A future `GameState` contract with explicit `enter`, `update`, `render`, and `exit` operations would make transitions easier to extend and test.

## Configuration-Driven Content

Positions, tuning values, room contents, rewards, and messages are externalised from the algorithms. This separates balancing/content changes from Java behaviour and avoids recompilation for every numeric adjustment.

The cost is string-based parsing at runtime. Missing or malformed keys can fail late, and domain meaning is spread between code and configuration. Production code should parse once into validated, typed configuration objects and provide actionable validation errors before the game loop starts.

## Explicit Collision Resolution

Collision detection uses a common bounds operation, while the active room applies consequences according to object type and state. Explicit branching made the interaction matrix easy to implement and inspect for a project with a bounded set of entities.

This approach becomes harder to maintain as combinations grow. A dedicated collision resolver, category/mask filtering, or double-dispatch design could separate interaction policy from room orchestration. Such a refactor should be driven by growth rather than introduced prematurely.

## Deterministic Frame-Based Timing

Cooldown counters and per-frame movement provide deterministic behaviour under the framework’s update model. They are easy to debug and sufficient for a fixed-rate educational game.

The limitation is dependence on frame rate. A production implementation should use delta time or a fixed simulation timestep so movement and cooldown duration remain stable under variable rendering performance.

## Maintainability Assessment

The design is strongest where abstractions correspond to real shared concepts. It is weakest where central coordination, rendering, mutation, and collision policy meet in the same room/controller methods. The appropriate evolution is targeted extraction rather than a wholesale rewrite:

1. Introduce typed game states and transition events.
2. Separate simulation updates from drawing.
3. Consolidate repeated room and projectile lifecycle behaviour.
4. Extract collision outcomes behind a small service.
5. Inject validated configuration rather than reading global properties.
6. Add tests around pure rules before changing architecture.

These steps preserve the working domain model while improving isolation, testability, and extension cost.

