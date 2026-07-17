# Resume Project Description

## Shadow Dungeon Game | Java, Bagel Framework, Object-Oriented Programming

- Built a room-based 2D action game in Java, implementing player controls, character abilities, enemy encounters, projectile combat, upgrades, collectibles, and multi-stage progression.
- Modelled reusable entity lifecycles with abstract enemy and projectile families, polymorphic behaviour, composition-based rooms, and a shared collision capability to keep gameplay responsibilities modular.
- Implemented normalised projectile targeting, waypoint-based enemy movement, frame-controlled cooldowns, safe projectile lifecycle removal, and collision outcomes across combatants, obstacles, hazards, doors, and items.
- Externalised gameplay tuning and level data to support maintainability, then evaluated the design for production improvements including explicit game states, typed configuration, isolated collision resolution, and automated testing.

## Short Version

- Developed a Java/Bagel 2D action game using polymorphic entity hierarchies, composition-based room management, collision detection, projectile targeting, and state-oriented progression.
- Designed reusable enemy and projectile lifecycles and configuration-driven gameplay systems, while identifying refactoring and testing strategies for a production-scale architecture.

## Interview Talking Points

- Why inheritance was limited to genuinely shared enemy/projectile lifecycles.
- Why rooms use composition and where room-controller responsibilities became too broad.
- How normalised direction vectors, reverse iteration, cooldowns, and collision ordering support stable combat.
- How the architecture could evolve toward explicit game states and independently testable systems.
- Why the complete source and course-provided resources are intentionally not public.

