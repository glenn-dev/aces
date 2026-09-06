# Aces — Game Design Overview

**Status:** Exploratory  
**Last reviewed:** 2026-09-06

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

## 21. Units, Profiles, Targeting, and Tile Occupancy

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
├── experience progress
├── rank
├── posture
└── runtime state
```

Every `UnitInstance` has a `MatchPolity` owner; that owner is not absent for neutral or autonomous units and is never an individual `User`. Operational control remains separate from ownership.

Health is an abstract percentage of remaining formation integrity or capacity. It does not encode the visual quantity of soldiers, vehicles, aircraft, or other represented elements. Presentation determines how a health percentage appears visually.

`Readiness` replaces the earlier ambiguous term `stamina` and represents operational sustainment such as fuel, general supplies, fatigue, and maintenance. `Ammo` is separate state representing specialized offensive capability.

### Target Classes

A unit's identity or family is distinct from the broader class used for targeting and combat interaction. A `UnitDefinition` and its `TargetClass` are therefore not assumed to be the same concept.

Exploratory examples of target classes include:

- infantry;
- light_ground;
- heavy_ground;
- air;
- naval;
- structure.

This vocabulary is not a frozen taxonomy. Its purpose is to let targeting and damage rules operate over meaningful broad categories without requiring a distinct hardcoded pairing for every combination of UnitDefinitions.

### Attack and Defense Profiles

For the initial ruleset, a `UnitDefinition` may be treated as having one primary attack profile. This is a simplifying assumption, not a permanent restriction; the model may support multiple attacks or weapons later if concrete gameplay needs justify them.

A primary attack profile may conceptually involve:

- base offensive power;
- minimum range;
- maximum range;
- allowed target classes.

Attack range uses a minimum and maximum. Illustratively, direct combat might use minimum 1 and maximum 1, while ranged artillery might use minimum 2 and maximum 3. These examples do not freeze the ranges of any unit.

Movement range and attack range are separate concepts and are not implicitly coupled.

A conceptual defense profile may provide baseline defensive resistance and interaction with TargetClass. Offensive capability and defensive resistance remain distinguishable concerns. Armor systems and damage types are intentionally deferred until a concrete design need demonstrates their value.

### Tile Occupancy

A tile may contain zero or more units.

```text
occupants: 0..n
```

A fixed `max_units` value is not currently assumed.

Whether a unit may enter or remain in a tile may instead depend on terrain, connection, unit definition, structure, events, ruleset, and other occupants.

A future interaction may therefore resemble:

```text
CanEnter(unit, tile)
```

Detailed unit taxonomy, capabilities, and balance remain outside the current checkpoint.

## 22. Readiness and Unit Posture

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

Low readiness progressively reduces combat effectiveness. This does not mean that every small reduction from maximum readiness must cause a meaningful combat penalty.

Readiness may conceptually be interpreted through ranges such as operational, degraded, critical, and exhausted. These are exploratory labels for derived conditions, not necessarily separate state stored in addition to readiness.

The intended behavior is that a healthy or normal readiness range produces little or no meaningful combat penalty, lower readiness progressively degrades combat effectiveness, and zero readiness represents exhaustion together with the severe operational constraints already described.

Exact readiness consumption, recovery, zero-readiness health degradation, effectiveness thresholds, and curves remain undefined.

`UnitPosture` is provisional runtime state of a `UnitInstance`, not a separate `UnitDefinition`. The current posture vocabulary is:

```text
UnitPosture
├── Maneuver
└── Defensive
```

`Maneuver` is the baseline operational posture. Conceptually, it provides normal incoming-damage behavior, normal or full movement capability subject to other rules, standard movement readiness cost, and higher passive or base readiness consumption relative to `Defensive`.

`Defensive` conceptually provides increased resistance to incoming damage, lower passive or base readiness consumption, reduced movement capability, increased readiness cost when moving, and potentially reduced offensive effectiveness.

No exact posture bonuses, penalties, percentages, or thresholds are defined. It also remains open whether posture may be selected after movement, at the beginning or end of an action window, or at another ruleset-defined point. Whether voluntarily attacking while `Defensive` changes posture or instead applies an offensive penalty is also unresolved.

## 23. Ammo and Fallback Combat

With ammo above zero, a unit uses its normal combat profile. With zero ammo, it may use fallback combat.

Fallback combat uses the base infantry unit's offensive capability as the reference floor. Base infantry is currently envisioned as the cheapest unit and the lowest base-damage reference in the game, but neither exact values nor the full unit catalog are frozen.

Fallback combat preserves normal range and TargetClass restrictions. Running out of ammo never allows a unit to target a class it could not normally target.

Initiating an offensive fallback attack consumes health and readiness. Defending at zero ammo remains possible and does not incur that additional offensive fallback cost.

If both ammo and readiness are zero, a unit may still initiate fallback combat when every other combat and targeting rule permits the attack. The offensive fallback health cost still applies. Any readiness cost is bounded by the zero floor and must not produce negative readiness.

Exact fallback damage, health cost, readiness cost, and related formulas remain undefined.

## 24. Combat Effectiveness and Retaliation

Combat effectiveness emerges from distinct contributors rather than from any one state value. The current conceptual contributors include:

- the UnitDefinition's attack and defense profiles;
- health;
- readiness;
- rank;
- posture;
- terrain;
- events and other contextual modifiers.

Their responsibilities remain separate:

- health represents remaining formation integrity or capacity;
- readiness represents operational sustainment such as fuel, supplies, fatigue, and maintenance;
- rank represents accumulated combat experience and veteran efficiency;
- posture represents the unit's current tactical operating mode;
- terrain, events, and context modify the circumstances in which the unit operates.

Health contributes progressively to combat effectiveness but is not identical to it. The agreed relationship is sublinear rather than strictly one-to-one: full health corresponds to full health-derived effectiveness, a unit at 50% health retains meaningfully more than 50% of its offensive capability, effectiveness still declines as health falls, and zero health means the unit is destroyed. This is intended to keep damaged units relevant for longer and reduce excessive snowballing. No mathematical curve or intermediate percentages are frozen.

Illustratively:

```text
effective attack
← base attack profile
← health factor
← readiness factor
← rank modifier
← posture modifier
← contextual modifiers

effective defense
← base defense profile
← health factor
← readiness factor
← rank modifier
← posture modifier
← contextual modifiers
```

This relationship is conceptual, not a final formula, and does not require the factors to combine multiplicatively.

Retaliation is currently a reactive consequence of an attack, not a separate voluntary action selected by the defending player:

```text
Attack
  ↓
primary damage to defender
  ↓
defender state changes
  ↓
evaluate retaliation eligibility
  ↓
if eligible, defensive-response damage to attacker
```

Retaliation occurs only when the defender survives the initial damage, the attacker is within the defender's effective attack range, the attacker belongs to a TargetClass the defender can normally target, and all other relevant rules permit the response. Targeting restrictions remain preserved.

The defender retaliates using its state after receiving the initial attack. A destroyed defender does not retaliate; damage sustained from the first strike may reduce the surviving defender's response effectiveness.

The same broader action-and-consequence model may later support other reactive effects, but defensive fire, mines, interception, fortifications, and similar possibilities are not current mechanics.

Exact attack, defense, damage, and retaliation calculations remain undefined.

## 25. Experience, Ranks, and Unit Merge

### Experience Progression

Any offensive or defensive combat interaction that causes more than zero damage grants one experience progress event or step. Destroying the target is not required. Under the initial ruleset, damage amount does not change the progress gained, and a qualifying defensive interaction grants the same progress as a qualifying offensive interaction unless later balancing changes this.

Experience progress and rank are separate concepts. Progress applies toward the next rank in an N-rank sequential hierarchy. When the next threshold is reached, the unit advances exactly one sequential rank and begins progress toward the following rank.

A rank may conceptually have sequential order, an experience threshold, and modifiers, without freezing a final rank-definition model. Exact thresholds remain undefined and ruleset-configurable.

The brainstorm's distinction between consecutive and lifetime combat progress does not drive active rank progression in the initial ruleset. Lifetime combat participation may still be retained as historical or statistical information.

### Rank Effects

Rank primarily represents veteran combat effectiveness. Higher rank primarily improves offensive and defensive effectiveness relative to the unit's own base profile; it does not grant absolute power unrelated to its `UnitDefinition`.

By default, rank does not increase maximum health, maximum readiness, maximum ammo, or movement range. More generally:

> Rank does not normally increase capacities; rank may improve efficiency.

Future design may explore small operational-efficiency benefits, such as slightly more efficient readiness consumption or subtle improvements related to logistics, resupply, repair, fallback costs, or comparable systems. These are possibilities rather than first-ruleset commitments. They should remain subtle rather than turning rank into a broad collection of large bonuses, and should not increase base resource capacities merely because a unit is more experienced.

No numeric rank modifiers are currently defined.

### Unit Merge

A merge represents consolidation of two compatible formations into one resulting `UnitInstance`.

For the initial ruleset, two units may merge only when they share the same `UnitDefinition`, have the same `MatchPolity` owner, occupy the same tile, and the `Ruleset` permits the merge:

```text
CanMerge(A, B)
```

This is conceptual validation, not an implementation interface. Heterogeneous combinations such as infantry with a tank are not part of the current merge model.

The resulting unit retains the shared `UnitDefinition`, `MatchPolity` owner, and tile. Its state accounts for the effective contribution of both source formations. Health, readiness, ammo, and experience are not blindly summed as percentages; for example, 70% readiness and 60% readiness do not become 130% readiness.

Merged health cannot exceed the normal maximum capacity of the shared `UnitDefinition` unless a future rule explicitly permits it. The handling of excess source health or capacity remains unresolved: it might be discarded, cause only a partial merge, cause the merge to be rejected, or remain as residual strength in a second unit.

Readiness and ammo are combined proportionally to the effective contribution of each source formation rather than added as raw percentages. Exact formulas remain open.

Experience that survives a merge is likewise proportional to how much of the resulting formation came from each source. A small veteran component does not automatically confer its full rank on a much larger inexperienced component, while meaningful veteran contribution should not simply disappear.

Conceptually, merge resolution may use:

```text
rank + progress
        ↓
temporary cumulative experience representation
        ↓ weighted by formation contribution
merged experience
        ↓
resolved back into rank + progress
```

The cumulative representation is a conceptual merge-resolution tool, not necessarily another persistent attribute of `UnitInstance`. Rank and progress remain the canonical experience state; no separate permanent veterancy value is currently introduced.

Exact health, readiness, ammo, and experience weighting formulas remain undefined.

## 26. Movement

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

## 27. Actions and World Mutation

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

## 28. Events

Events remain part of the conceptual model but are not yet deeply specified.

The brainstorm currently distinguishes natural events and artificial events generated by players, teams, or other systems.

Events may affect tiles, units, structures, settlements, Polities, teams, connections, and environmental state.

Events may create temporary or lasting world mutations.

Their exact lifecycle and scope remain open.

## 29. Current Open Questions

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
- final TargetClass taxonomy;
- exact attack, defense, damage, and retaliation calculations;
- exact health-effectiveness curve;
- exact readiness-effectiveness curve and thresholds;
- readiness consumption, recovery, and zero-readiness health degradation;
- ammo consumption and replenishment;
- fallback combat costs and damage formulas;
- exact experience thresholds and rank modifiers;
- exact operational-efficiency benefits from rank, if any;
- merge formulas and excess health or capacity behavior;
- exact readiness and ammo weighting during merge;
- exact experience weighting during merge;
- resulting posture and treatment of other runtime state during merge;
- posture-change timing and exact `Defensive` bonuses and penalties;
- whether voluntarily attacking while `Defensive` changes posture;
- whether and when multiple attacks or weapons are introduced;
- damage types, if a concrete gameplay need emerges;
- resources and economy;
- logistics and communication networks;
- event lifecycle;
- fog of war and information visibility;
- save, replay, simulation, and agent interaction concerns.

These questions should be resolved progressively through game-design iteration rather than prematurely frozen.

## 30. Current Design Direction

```text
Persistent Definitions
├── User
├── PolityDefinition
├── MapDefinition
├── StructureDefinition
└── UnitDefinition
    ├── target class
    ├── primary attack profile (initially)
    ├── defense profile
    └── movement / capacity-related definition

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
│   │   ├── owner: MatchPolity
│   │   ├── position
│   │   ├── health
│   │   ├── readiness
│   │   ├── ammo
│   │   ├── experience progress
│   │   ├── rank
│   │   ├── posture
│   │   └── runtime state
│   └── Events
├── TurnState
├── Ruleset
└── Outcome
```

Merge remains an interaction between compatible UnitInstances that produces one consolidated UnitInstance, rather than a new persistent entity in this structural summary.

The central design principle remains:

> define explicit entities and state, let actions and events produce observable consequences, and allow the world to evolve as a result.
