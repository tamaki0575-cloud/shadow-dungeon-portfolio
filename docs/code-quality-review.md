# Code Quality Review

## Review Scope

This review is based on the local implementation while deliberately omitting source excerpts, assessment instructions, configuration values, and course-provided material. It evaluates engineering qualities relevant to an internship or graduate portfolio.

## Strengths

### Domain concepts are visible

Classes map clearly to concepts such as player, enemy, projectile, room, door, key, treasure, store, obstacle, and UI. This reduces the mental translation required to understand the gameplay model.

### Reusable entity lifecycles

Common enemy and projectile behaviour is centralised instead of duplicated across every variant. Polymorphic collections allow room logic to update multiple variants through shared contracts.

### Appropriate use of composition

Rooms own the objects active within them, matching runtime lifecycle and encounter scope. This is preferable to storing all entities directly in the main game coordinator.

### Consistent collision capability

A shared overlap contract makes geometric checks available across otherwise unrelated object types. Axis-aligned bounds are an appropriate balance of simplicity and accuracy for the game’s visual style.

### Safe mutation during iteration

Transient projectiles are processed in reverse when removals can occur, avoiding index shifts that would skip elements or produce invalid access.

### Externalised tuning

Gameplay values and layout data are separated from core algorithms, making balancing changes easier and reducing hard-coded numeric data in behaviour classes.

### Evidence of iterative development

The local history reflects incremental implementation of entities, combat, rooms, and documentation rather than one monolithic change. Submission metadata itself remains private and is not part of this portfolio repository.

## Weaknesses and Risks

### Centralised global state

The main coordinator relies heavily on globally accessible state and transition operations. This makes dependencies implicit, complicates isolated tests, and increases the chance that one room mutates another through shared state.

### Large room/controller responsibilities

The battle-room component coordinates entity creation, updates, rendering, collisions, rewards, drops, completion, and doors. These are related, but the concentration makes changes riskier and encourages long methods.

### Update and rendering are interleaved

Many flows update and draw within the same operation. This limits headless testing, makes pause/snapshot behaviour more complex, and can cause partially updated frames around transitions.

### Repeated room behaviour

Preparation, battle, and end contexts repeat parts of projectile advancement, boundary checks, rendering, and transition guards. Duplication increases the chance that a bug fix is applied to only one room type.

### Type-based collision branching

Collision outcomes use explicit object-type checks. The approach is understandable for the current scope but becomes increasingly fragile as new combinations are added.

### String-based configuration

Runtime properties and room identifiers are flexible but not type-safe. Errors can surface only after launch, and duplicate or inconsistent keys are difficult for the compiler to detect.

### Build configuration inconsistency

The local build descriptor contains conflicting Java compiler-version declarations and an unrelated artifact name. A private archival repository should normalise these values, but the build file is intentionally excluded from this public documentation repository.

### Limited automated verification

The implementation is strongly coupled to framework images, input, and global properties, and no automated test suite is present. Manual gameplay testing alone makes regressions harder to detect.

## Production-Quality Improvements

### 1. Introduce explicit game states

Define a small lifecycle contract for preparation, battle, end, and game-over states. A controller would own only the current state and process requested transitions, reducing conditional and static coupling.

### 2. Split simulation from presentation

Run domain updates first and render the resulting state afterward. This enables deterministic tests, clean pause behaviour, replay/snapshot support, and clearer transition boundaries.

### 3. Extract room-shared lifecycle behaviour

Create a shared room foundation or composed entity collection for common player, projectile, boundary, and render ordering. Keep encounter-specific completion and activation rules in battle-focused code.

### 4. Isolate collision policy

Move collision outcomes into a resolver with explicit categories. Detection should report candidate contacts; resolution should apply damage, blocking, collection, destruction, or transition rules.

### 5. Use typed configuration

Parse external values once during startup into immutable records or value objects. Validate required fields, numeric ranges, duplicate identifiers, and valid destinations before the game begins.

### 6. Use time-based simulation

Replace frame-dependent movement and cooldowns with delta-time values or a fixed simulation timestep. This prevents gameplay speed from depending on rendering performance.

### 7. Improve diagnostics

Replace silent parsing or empty default branches with contextual errors and structured logging during initialisation. Fail fast when a room refers to an unknown entity or destination.

## Suggested Testing Strategy

### Unit tests

- Direction normalisation and constant projectile speed.
- Cooldown boundaries and allowed firing frames.
- Damage, death, reward, key, and purchase invariants.
- Patrol waypoint advancement and route wrapping.
- Axis-aligned overlap boundary cases.
- Destructible-object rewards occurring exactly once.
- Configuration parsing and validation failures.

### State-transition tests

- Entering and leaving each room transfers player ownership correctly.
- A previous room stops updating after a transition request.
- Battle completion unlocks exits exactly once.
- Restart reconstructs initial player and room state.
- Transient projectiles do not leak between rooms.

### Integration tests

- Complete progression from preparation through both encounters to the ending.
- Character-specific environmental and reward behaviour.
- Enemy defeat, key drop, key collection, and treasure consumption.
- Store purchases with sufficient and insufficient currency.

### Testability enablers

- Inject input commands rather than requiring live keyboard state.
- Wrap image bounds behind a lightweight collision-shape interface.
- Inject configuration and random/time sources.
- Keep update rules independent from rendering wherever possible.

## Recruiter-Facing Presentation Improvements

- Keep this repository documentation-only unless written publication permission is obtained.
- Add authorised screenshots or a short demo only after asset rights are confirmed.
- Use the architecture diagram to explain boundaries and trade-offs in interviews.
- Describe state-oriented design accurately rather than claiming a formal State pattern.
- Discuss the largest refactoring opportunities and why they were not necessary for the original scope.
- Pair this case study with at least one fully public, independently designed project that recruiters can build and inspect.

