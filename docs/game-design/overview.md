# Aces — Game Design Overview

**Status:** Exploratory  
**Last reviewed:** 2026-09-05

> This document captures the current conceptual model of Aces.
> It is a working design checkpoint, not a frozen specification.
> Detailed mechanics, formulas, balance values, naming, and implementation choices remain subject to iteration.

## 1. Purpose of This Document

This document describes the current game-design model that has emerged from early brainstorming and design discussions.

It exists to preserve the concepts and decisions that are already coherent enough to be useful, while keeping unresolved questions explicit.

The durable purpose and principles of the project belong in `genesis.md`. Raw, unfiltered ideation belongs in `brainstorm.md`.

Within this document, statements expressed as the current model are agreed conceptual decisions. Provisional assumptions identify simplifying rules intended for an initial ruleset, while open questions identify details that have deliberately not been settled.

## 2. Core Runtime Model

Aces is currently conceived as a strategy game where participants act on entities inside an explicit world state.

At a high level:

> actor → target → action → validation → consequence → new state

Actions may include movement, combat, construction, destruction, capture, repair, terrain modification, or other interactions defined later by the ruleset.

The first implementation should remain concrete and game-focused. Generalized abstractions should only be extracted after they prove useful in the game itself.

## 3. Persistent Definitions vs Runtime State

Aces should distinguish persistent definitions from ephemeral runtime instances.

Conceptually:

```text
persistent definition
        ↓ instantiate
runtime instance
        ↓ actions / events
mutable state
```

Examples:

```text
PolityDefinition
        ↓
MatchPolity
```

```text
MapDefinition
        ↓
World
```

The persistent definition describes what something is intended to be at the start.

The runtime instance describes what it currently is inside a specific match.

## 4. User

`User` is a persistent product identity.

A user is not globally defined only as a player, spectator, moderator, or administrator. Roles and permissions should depend on participation context.

Conceptually:

```text
User
├── identity
├── profile
└── participations
```

A user may participate differently across matches.

Exact role vocabulary remains open.

## 5. Polity

The current domain term for the persistent definition of an organized in-world entity is `Polity`.

A Polity may later be presented thematically as a republic, country, state, faction, clan, corporation, or another concept.

Conceptually:

```text
PolityDefinition
├── identity
├── name
├── demonym
├── flag
├── emblem
├── motto
├── anthem
├── type
├── tags
└── other configuration
```

A `PolityDefinition` is persistent.

A match creates an ephemeral instance based on that definition:

```text
PolityDefinition
        ↓ instantiate
MatchPolity
```

`MatchPolity` contains mutable state that only makes sense within the match, such as resources, territory, units, structures, settlement relationships, alliances, and current status.

A `MatchPolity` may represent either a human-controlled or an NPC-controlled participant. A match may contain one or more NPC-controlled MatchPolities, and a ruleset may distinguish several such Polities when different neutral or autonomous actors should remain separate. This does not require a universal `NeutralPolity` shared by every match.

A provisional concept such as control mode may help express this distinction, but its name and relationship to participation and automation remain exploratory.

The exact final names of these entities may still evolve.

## 6. Team

`Team` is currently conceived as an entity that exists only within a match.

It represents a temporary alignment between one or more match Polities.

```text
Team
└── MatchPolities [1..n]
```

This keeps identity separate from alliance:

```text
Polity → persistent identity
Team   → match-specific alignment
```

## 7. User Participation

Rather than coupling users directly to teams and Polities through ad hoc relationships, the current model introduces a participation concept.

```text
UserParticipation
├── user
├── match
├── role
├── team?
├── match_polity?
└── permissions
```

This allows identity, alignment, role, and permission to remain separate concepts.

Exact permissions and roles remain open.

## 8. Ownership vs Control

Ownership and operational control are different concepts.

Owned runtime entities retain a `MatchPolity` owner regardless of how that Polity is controlled:

```text
UnitInstance.owner      = MatchPolity
StructureInstance.owner = MatchPolity
```

Neutral or autonomous units and structures should normally belong to an NPC-controlled `MatchPolity`, rather than representing neutrality through a missing owner. Multiple distinct NPC-controlled MatchPolities may coexist when the ruleset calls for them; no mandatory universal neutral singleton is assumed.

For example:

```text
MatchPolity X owns Unit 17
User A controls Unit 17
```

A user may gain, lose, delegate, or share operational responsibility without changing ownership of the entity.

A possible future concept is:

```text
ControlAssignment
├── participant
├── entity_or_scope
├── validity
└── permissions
```

Whether human and NPC control ultimately use the same control-assignment model remains open. No final controller schema is implied here.

The original idea that only one player may manage a tile at a time is therefore not currently treated as a fundamental rule.

## 9. Match

`Match` is the runtime container of a game session.

Conceptually:

```text
Match
├── ruleset
├── world
├── teams
├── match_polities
├── user_participations
├── control_assignments
├── turn_state
├── active_events
├── status
└── outcome
```

A match exists before gameplay begins because it may need to be configured.

A possible lifecycle is:

```text
draft
  ↓
ready
  ↓
active
  ↓
finished
```

Additional states remain open.

## 10. Ruleset

A match should not hardcode every game rule directly.

A `Ruleset` is the current conceptual place for configurable game rules.

Possible areas include:

```text
Ruleset
├── victory conditions
├── turn rules
├── allowed entities
├── movement rules
├── combat rules
├── economy rules
└── event rules
```

This remains conceptual.

The goal is not to build a generic rules engine prematurely, but to avoid coupling all gameplay decisions directly to the `Match` entity.

## 11. Turn Ownership

The original brainstorm proposed turns by republic.

Under the current model, the more precise concept is that a turn may belong to a `MatchPolity`, not directly to an individual user.

```text
TurnOwner = MatchPolity
```

If multiple users manage the same Polity, they may share the same action window while control assignments determine who may operate which entities.

Alternative turn models remain open.

## 12. Map and World

Aces distinguishes the spatial definition of the scenario from its mutable runtime state.

### MapDefinition

`MapDefinition` describes the initial spatial topology.

```text
MapDefinition
├── tiles
├── connections
├── bounds
└── metadata
```

It should not contain runtime units, active events, temporary damage, or current control.

### World

`World` represents the current mutable state of that map inside a match.

```text
World
├── tile states
├── connection states
├── structure instances
├── settlements
├── unit instances
├── environmental state
├── active events
└── control state
```

For the initial design:

```text
Match → one World
World → one MapDefinition
```

Support for multiple worlds or maps in a single match is intentionally deferred.

## 13. Hexagonal Map

The world uses a hexagonal tile grid.

Axial coordinates are the current preferred representation:

```text
(q, r)
```

The coordinate/topology of a tile is expected to remain stable during a match.

The physical state of the tile may change.

## 14. Tile Definition and Tile State

A tile should distinguish stable identity from mutable geographic state.

Conceptually:

```text
HexTileDefinition
├── coordinate
└── initial state
```

Runtime:

```text
TileState
├── elevation
├── vegetation
├── current
├── occupants
├── primary occupiable element?
├── controller?
├── active events
└── temporary modifiers
```

The exact attribute model remains exploratory.

## 15. Elevation, Land, Water, and Depth

`land` and `water` should not currently be stored as independent primary tile types.

Instead, surface classification may be derived from elevation relative to the world's sea level.

Conceptually:

```text
WorldState
└── sea_level
```

```text
TileState
└── elevation
```

Derived:

```text
surface_class =
    elevation > sea_level ? land : water
```

Depth may also be derived for submerged tiles:

```text
depth =
    max(0, sea_level - elevation)
```

The exact behavior when `elevation == sea_level` remains undefined.

`current` remains an independent attribute because it cannot be trivially derived from elevation alone.

## 16. Mutable Terrain

Terrain is intended to be mutable during a match.

Player actions, team actions, natural events, or other systems may eventually modify properties such as elevation, vegetation, surface classification, passability, or temporary environmental state.

Examples may eventually include excavation, filling, deforestation, flooding, erosion, or other terrain transformations.

The model should support this possibility now.

The detailed mechanics, costs, eligible units, time requirements, and restrictions are intentionally deferred.

The topology of the hex grid itself is currently assumed to remain stable.

## 17. Connections

Movement between neighboring tiles occurs through their shared boundary.

A connection should therefore be modeled as a first-class concept rather than as a simple property of a tile.

```text
HexTile A
     │
 Connection
     │
HexTile B
```

Conceptually:

```text
ConnectionDefinition
├── tile_a
├── tile_b
└── initial state
```

Runtime:

```text
ConnectionState
├── passability
├── infrastructure
├── damage
├── blockage
└── temporary modifiers
```

Connections may also change during a match.

## 18. Roads and Communication Network

Connections may contain roads or other infrastructure.

The original concept describes roads with levels that may influence transitability, movement efficiency, capacity, and exposure.

A provisional conceptual model is:

```text
Route
├── level
├── mobility modifier
├── capacity
└── exposure
```

No numeric meaning is currently frozen.

Over time, these routes may form a broader communications or logistics network.

## 19. Structures

Structures preserve the distinction between persistent definition and runtime state:

```text
StructureDefinition
        ↓ instantiate
StructureInstance
```

A `StructureDefinition` describes the configurable identity, capabilities, placement conditions, and available levels of a kind of structure. A `StructureInstance` is an entity in a match that may be built, operated, damaged, repaired, captured, upgraded, disabled, and destroyed.

Structure levels are not globally fixed at exactly three. A definition may support N sequential or otherwise configured levels. The effects, requirements, and limits of those levels remain ruleset concerns.

The initial exploratory structure families are:

- Base;
- Port;
- Airport;
- Production;
- Research.

These families are a starting catalog, not a final taxonomy.

Every `StructureInstance` has a `MatchPolity` owner; that owner is not absent for neutral or autonomous structures and is never an individual `User`. As with units, ownership and operational control remain separate concepts.

For an initial ruleset, a tile has at most one primary occupiable element. A structure and a settlement therefore currently compete for the same slot:

```text
TileState
└── primary occupiable element: 0..1
    ├── StructureInstance
    └── Settlement
```

This is a provisional simplifying constraint, not a permanent claim that settlement and infrastructure must always exclude one another.

Placement should be validated through the relationship among the proposed structure, world state, and rules. Structure family may contribute conditions, but should not be the only conceptual source of placement validity.

A Port remains spatially special: it must relate to a specific land-water `Connection`, rather than merely occupying an unspecified position within a land tile.

The runtime lifecycle may include all of the following without requiring one fixed sequence:

```text
build
operate
damage / repair
upgrade / disable
capture
destroy
```

Capture and destruction are separate actions and concepts. Destroying a structure also need not immediately remove it from the world; persistent residual states such as ruins remain an open possibility.

Structures may also be affected by combat and events.

Detailed construction rules, costs, family capabilities, production behavior, level effects, capture rules, and residual-state behavior remain for later design.

## 20. Settlements

`Settlement` is a distinct domain concept from `Structure`.

The initial exploratory settlement levels are:

```text
Hamlet → Town → City → Metropolis
```

These labels establish an initial progression vocabulary, while exact thresholds and effects remain undefined.

A settlement may have its own identity and development state. It should conceptually develop or degrade rather than necessarily being built or destroyed in exactly the same way as infrastructure. A devastated settlement need not cease to exist.

Settlement acceptance is relational by Polity, not a single value intrinsic to the settlement:

```text
Acceptance(settlement, polity) → value from 0 to 100
```

A settlement may therefore hold different acceptance values toward multiple Polities at the same time. Settlement ownership, tile control, and acceptance are separate concepts; none should be inferred automatically from another.

Capital status is not a hardcoded settlement subtype. It belongs to a `MatchPolity` relationship governed by the match's `Ruleset`.

Under the provisional single-slot rule described above, a Settlement and a Structure cannot occupy the same tile as primary elements. The design may later evolve toward separate settlement and infrastructure occupancy if gameplay requires it.

## 21. Units and Tile Occupancy

Units preserve the distinction between persistent definition and runtime state:

```text
UnitDefinition
        ↓ instantiate
UnitInstance
```

A `UnitInstance` conceptually includes:

```text
UnitInstance
├── definition
├── owner: MatchPolity
├── position
├── health
├── readiness
├── ammo
├── experience progress / steps
├── rank
└── runtime state
```

Every `UnitInstance` has a `MatchPolity` owner; that owner is not absent for neutral or autonomous units and is never an individual `User`. Operational control remains separate from ownership.

Health is an abstract percentage of remaining integrity or combat capacity. It does not encode the visual quantity of soldiers, vehicles, aircraft, or other represented elements. Presentation determines how a health percentage appears visually.

Effectiveness is not identical to health. Health may contribute to contextual effectiveness alongside readiness, experience, rank, terrain, events, and other ruleset-defined factors.

`Readiness` replaces the earlier ambiguous term `stamina`. Readiness abstracts fuel, general supplies, fatigue, and maintenance. `Ammo` is separate state representing specialized offensive capability.

A tile may contain zero or more units.

```text
occupants: 0..n
```

A fixed `max_units` value is not currently assumed.

Whether a unit may enter or remain in a tile may instead depend on terrain, connection, unit type, structure, events, ruleset, and other occupants.

A future interaction may therefore resemble:

```text
CanEnter(unit, tile)
```

Detailed unit taxonomy, capabilities, and balance remain outside the current checkpoint.

## 22. Readiness

Readiness has a floor of zero:

```text
readiness >= 0
```

The provisional first-ruleset behavior is:

- At readiness above zero, normal movement may occur, subject to all other movement rules.
- At zero readiness, a unit cannot move from one tile to another.
- A unit progressively loses health for each turn it remains at zero readiness.
- A unit at zero readiness may still attack an immediately adjacent valid target.
- A unit at zero readiness may still defend.

Exact readiness consumption, recovery, health degradation, timing, and other balance formulas remain undefined.

## 23. Ammo and Fallback Combat

With ammo above zero, a unit uses its normal combat profile. With zero ammo, it may use fallback combat.

Fallback combat uses the base infantry unit's offensive capability as the reference floor. Base infantry is currently envisioned as the cheapest unit and the lowest base-damage reference in the game, but neither exact values nor the full unit catalog are frozen.

Fallback combat preserves all normal targeting restrictions. Running out of ammo never allows a unit to target a class it could not normally target.

Initiating an offensive fallback attack consumes health and readiness. Defending at zero ammo remains possible and does not incur that additional offensive fallback cost.

If both ammo and readiness are zero, a unit may still initiate fallback combat when every other combat and targeting rule permits the attack. The offensive fallback health cost still applies. Any readiness cost is bounded by the zero floor and must not produce negative readiness.

Exact fallback damage, health cost, readiness cost, and related formulas remain undefined.

## 24. Experience and Ranks

Any offensive or defensive combat interaction that causes more than zero damage grants experience progress. Destroying the target is not required.

For the current provisional rule, each qualifying combat interaction grants one experience step or progress event regardless of the amount of damage caused. Damage-weighted experience remains a possible future alternative, not a current rule.

Defensive combat currently grants no bonus beyond the normal progress for a qualifying combat interaction. A defensive bonus may be reconsidered during balancing.

Experience progress and rank are separate concepts. The rank hierarchy supports N ranks, and progression through that hierarchy is strictly sequential. Exact thresholds and rank modifiers remain undefined.

The brainstorm's distinction between consecutive and lifetime combat progress remains an open design question rather than a frozen rule. Experience transfer when units merge is also unresolved.

## 25. Movement

Movement should be evaluated through the relationship between the moving unit, the connection crossed, the destination tile, and current world state.

Conceptually:

```text
Unit
  ↓
CanTraverse(connection, destination)
  ↓
allowed?
cost
exposure
effects
```

Movement cost may eventually depend on terrain interaction, connection modifiers, events, and unit-specific modifiers.

No formula is currently frozen.

## 26. Actions and World Mutation

Combat should not be treated as the only meaningful state-changing action.

A future action vocabulary may include:

```text
Move
Attack
Build
Destroy
ModifyTerrain
Capture
Repair
...
```

All can follow the same general interaction model:

```text
actor
  ↓
target
  ↓
action
  ↓
validation
  ↓
cost / consequence
  ↓
new world state
```

This is both a gameplay model and one of the broader abstractions Aces intends to explore.

## 27. Events

Events remain part of the conceptual model but are not yet deeply specified.

The brainstorm currently distinguishes natural events and artificial events generated by players, teams, or other systems.

Events may affect tiles, units, structures, settlements, Polities, teams, connections, and environmental state.

Events may create temporary or lasting world mutations.

Their exact lifecycle and scope remain open.

## 28. Current Open Questions

The following areas remain intentionally unresolved:

- final role vocabulary for users;
- exact semantics of moderation and administration;
- final naming around Polity runtime instances;
- NPC behavior, turn participation, resources, diplomacy, automation, and the eventual controller model;
- team size and composition limits;
- turn ownership and whether turns are sequential or phase-based;
- victory-condition model;
- exact ruleset boundaries;
- tile behavior exactly at sea level;
- terrain-editing mechanics and costs;
- final structure taxonomy and the capabilities of each initial family;
- construction, repair, upgrade, disablement, capture, and destruction mechanics;
- structure level configurations and progression rules;
- whether and how destroyed structures persist as ruins or other residual states;
- whether settlement and infrastructure should eventually have separate occupancy;
- settlement identity fields, development and degradation rules, and level effects;
- how settlement acceptance changes and what consequences it has;
- how capitals are designated and what their capture or loss means under a ruleset;
- connection infrastructure types;
- capacity and stacking rules for units;
- final unit taxonomy and the base infantry reference profile;
- combat targeting, effectiveness, and damage formulas;
- readiness consumption, recovery, and zero-readiness health degradation;
- ammo consumption and replenishment;
- fallback combat costs and damage formulas;
- experience thresholds, rank modifiers, and whether progress is consecutive or lifetime-based;
- experience transfer when units merge;
- resources and economy;
- logistics and communication networks;
- event lifecycle;
- fog of war and information visibility;
- save, replay, simulation, and agent interaction concerns.

These questions should be resolved progressively through game-design iteration rather than prematurely frozen.

## 29. Current Design Direction

```text
Persistent Definitions
├── User
├── PolityDefinition
├── MapDefinition
├── StructureDefinition
└── UnitDefinition

Runtime Match
├── MatchPolities (human- or NPC-controlled)
├── Team
├── UserParticipation
├── ControlAssignment
├── World
│   ├── TileState
│   ├── ConnectionState
│   ├── StructureInstances
│   ├── Settlements
│   ├── UnitInstances
│   └── Events
├── TurnState
├── Ruleset
└── Outcome
```

The central design principle remains:

> define explicit entities and state, let actions and events produce observable consequences, and allow the world to evolve as a result.
