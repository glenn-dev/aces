# Aces — Game Design Overview

**Status:** Exploratory  
**Last reviewed:** 2026-09-07

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

## 11. Turn Ownership and Unit Activation

The original brainstorm proposed turns by republic.

Under the current model, a turn belongs conceptually to a `MatchPolity`, not to a `User`, `Team`, or `UnitInstance`.

```text
TurnOwner = MatchPolity
```

Conceptually:

```text
Match
  ↓
active MatchPolity
  ↓
eligible UnitInstances
  ↓
Unit Activations
```

Users operate entities through the participation, permissions, and control relationships already described. User identity does not own the game turn, and changing control between users does not create a new turn or refresh a unit. This separation supports human-controlled and NPC-controlled MatchPolities as well as future delegated or shared control models.

During its turn, a `MatchPolity` may activate eligible `UnitInstances` in an order permitted by the `Ruleset`. Each unit normally has one activation opportunity per turn of its `MatchPolity`. A Unit Activation is the operational opportunity of that formation during that turn:

```text
Unit Activation
├── optional Posture change
├── optional Movement
├── optional Primary Action
└── End Activation
```

This diagram describes available conceptual capabilities, not a mandatory fixed execution order. A Unit Activation is not a generic Action Point pool, and no numeric Action Points are introduced.

A unit normally cannot begin a second activation during the same `MatchPolity` turn unless an explicit future rule permits it. Its activation eligibility resets when the appropriate next turn for its `MatchPolity` begins, not when operational control changes between users.

### Movement and Primary Action

Movement and Primary Action are distinct, optional parts of a Unit Activation. Movement represents traversal of a valid path and may cross multiple neighboring tiles and Connections. It remains governed by movement capability, Readiness, Posture, world state, Connections, movement cost, and `Ruleset` constraints; it is not a sequence of generic action points.

A Primary Action is the unit's principal tactical action and may normally be consumed at most once per activation. The currently known Primary Actions are Attack and Merge. Other Primary Actions may be introduced later, but are not current mechanics.

A unit may voluntarily end its activation without using its available Movement, Primary Action, or Posture change opportunity.

### Sequencing and End Activation

Movement does not automatically end an activation. After completing valid Movement, a unit may retain its Primary Action when that specific action and the `Ruleset` permit it. Each Primary Action may define its own sequencing restrictions; this does not establish a universal action-sequencing system.

The default Attack sequence permits Movement followed by Attack when all movement, targeting, combat, and `Ruleset` constraints are satisfied:

```text
Movement → Attack → End Activation
```

Attack followed by Movement is not permitted by default. Attack normally ends the Unit Activation, so a unit that attacks before moving does not then gain a movement opportunity.

Merge is also a Primary Action, so Attack and Merge cannot normally both occur during one activation. Its specific sequencing and effects on participating units are refined in the Unit Merge section.

Once a Primary Action has ended an activation, the unit cannot subsequently move, change Posture, perform another Primary Action, or voluntarily reopen that activation. This does not require every possible future Primary Action to end activation; future actions may define different semantics.

A unit may also explicitly end its activation and forgo any unused opportunities. Once completed, the activation cannot normally be reopened, and the unit cannot normally activate again during the same `MatchPolity` turn. Only an explicit future `Ruleset` rule may override this.

More generally, no operation should improve current-turn action eligibility merely by transforming, merging, transferring, or otherwise changing a `UnitInstance`:

> state or resource changes do not restore spent action opportunities.

The complete match turn lifecycle, higher-level ordering, and exact action-state representation remain open.

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

Spatial distance should be reasoned about as hex-tile or graph distance under the map topology, rather than as Euclidean or presentation-space distance. Conceptually:

```text
origin tile
    ↓
adjacent tiles
    ↓
successive hex-distance rings
```

A tile's spatial distance from another tile is therefore the number of neighboring hex-grid steps that separate them under that topology. Axial coordinates identify tiles; they do not by themselves define a movement cost or whether a destination is reachable.

The exact canonical distance formula, coordinate library, and other implementation choices remain intentionally unfrozen.

The coordinate and topology of a tile are expected to remain stable during a match. Normal hex distance between two coordinates may therefore remain stable as well.

The physical state of tiles and connections may change, so stable hex distance does not imply stable reachability or movement cost.

Hex-distance rings may later prove useful for area of effect, visibility, detection, influence, communication, event radius, or other spatial mechanics. These are future design possibilities, not current mechanics or new domain entities.

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

Attack range uses conceptual `min_range` and `max_range` values expressed as valid hex distances from the attacker's tile. A target is spatially eligible only when its hex distance from the attacker falls within the permitted range, subject to every other targeting, combat, and Ruleset constraint.

Illustratively:

- `min_range = 1`, `max_range = 1` describes adjacent tiles only;
- `min_range = 2`, `max_range = 3` describes tiles in the second and third hex-distance rings.

These are examples only and do not assign ranges to any concrete `UnitDefinition`. Conceptually, a range may describe one distance ring, several consecutive rings, or another Ruleset-defined subset of otherwise valid tiles. This does not imply a generalized spatial-query system.

Range and targeting remain independent dimensions of attack eligibility:

```text
spatially in range
AND valid TargetClass
AND applicable combat and Ruleset constraints
→ target may be attackable
```

Being inside the valid attack range is necessary but does not by itself make the target attackable.

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

No exact posture bonuses, penalties, percentages, or thresholds are defined.

Changing Posture is an operational choice within a Unit Activation, not a Primary Action, and therefore does not itself consume the unit's Primary Action. By default, a unit may change Posture at most once during its activation unless a future `Ruleset` explicitly permits otherwise. This prevents repeated switching such as `Defensive → Maneuver → Defensive` merely to obtain advantages of both states. No numeric cost is currently assigned; a future `Ruleset` may impose a Readiness cost or another restriction.

The Posture relevant to an operation is the Posture held when that operation occurs. Movement performed while `Defensive` uses Defensive movement semantics and costs; changing to `Maneuver` afterward does not retroactively alter that Movement. An Attack performed while `Defensive` uses the applicable Defensive offensive penalty and does not itself change the unit's Posture. Defensive does not absolutely prohibit Movement followed by Attack; its established movement, Readiness, and offensive consequences apply subject to `Ruleset` refinement.

Illustrative sequences include:

```text
Defensive
  → change to Maneuver
  → Movement
  → Attack
  → activation ends in Maneuver

Defensive
  → Movement under Defensive semantics
  → Attack under Defensive offensive semantics
  → activation ends in Defensive

Maneuver
  → Movement
  → change to Defensive
  → End Activation
```

These examples are not mandatory workflows.

Posture persists across activations and turns. A unit does not automatically reset to `Maneuver` when its activation ends, its `MatchPolity` turn ends, another `MatchPolity` becomes active, or its next activation begins. Ending an activation in `Defensive` therefore leaves Defensive relevant during subsequent enemy activity, including incoming combat. Passive or current Posture effects use the unit's persistent current Posture.

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

Retaliation is currently a reactive consequence of an attack, not a separate voluntary action selected by the defending player, Primary Action, or Unit Activation:

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

Retaliation occurs only when the defender survives the initial damage, the attacker's hex distance falls within the defender's valid attack range, the attacker belongs to a TargetClass the defender can normally target, and all other relevant rules permit the response. Targeting restrictions remain preserved independently from spatial range.

The defender retaliates using its state after receiving the initial attack. A destroyed defender does not retaliate; damage sustained from the first strike may reduce the surviving defender's response effectiveness.

Retaliation does not consume a separate activation opportunity under the current conceptual model.

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

A Merge is an explicit, directed operational action that consolidates one compatible formation into another:

```text
source UnitInstance
        ↓ Merge
target UnitInstance
```

The target is the continuing formation. It retains its runtime identity rather than Merge normally creating a third unit through symmetric `A + B → new C` semantics. A future `Ruleset` could explicitly introduce different semantics, but they are not part of the current model.

Merge eligibility remains conceptual validation rather than an implementation interface. The source and target must share the same `UnitDefinition`, have the same `MatchPolity` owner, occupy the same tile, and be permitted to merge by the `Ruleset`. Heterogeneous combinations such as infantry with a tank are not part of the current Merge model.

The target may absorb source contribution only up to its normal capacities. Merge must not implicitly create over-capacity state. For health:

```text
target.health <= target.max_health
```

If the target cannot absorb the entire source formation, the Merge may be partial. The amount actually incorporated into the target determines the source contribution that was absorbed. Exact arithmetic and rounding remain undefined.

Two conceptual remainder policies govern what happens to an unabsorbed source contribution. Their working names do not freeze implementation vocabulary:

- `KEEP_REMAINDER` is the default. The target absorbs only what it can receive. If source contribution remains, the source `UnitInstance` continues to exist with its own runtime identity and posture, together with the proportional composition, resources, experience, and other state associated with the remaining formation.
- `DISBAND_REMAINDER` is an explicit player decision, not an automatic threshold behavior. The target absorbs the health contribution it can receive, after which the remaining source formation is deliberately disbanded and the source `UnitInstance` ceases to exist. No source-health threshold automatically selects this policy.

Health, Readiness, and Ammo retain different meanings during Merge:

- Health represents formation integrity and remaining capacity. The target receives health only up to its normal maximum. Under `KEEP_REMAINDER`, unabsorbed health remains with the source. Under `DISBAND_REMAINDER`, unabsorbed source health is lost when the remainder is disbanded.
- Readiness represents general operational preparedness, including abstractions such as fuel, general supplies, fatigue, maintenance, and operational condition. It remains separate from Ammo.
- Ammo represents specialized offensive capability and ammunition availability. It remains separate from Readiness.

Under `KEEP_REMAINDER`, Readiness and Ammo contributions transfer proportionally with the part of the source formation actually absorbed. The surviving source retains the corresponding contribution associated with its remaining formation.

`DISBAND_REMAINDER` intentionally has different resource semantics. All transferable specialized ammunition from the source becomes available to the target, as does all transferable or recoverable operational contribution represented through Readiness. This describes recovery and consolidation of operational contribution and resources; it does not imply that literal fatigue or maintenance state moves physically between formations. Both attributes remain bounded by the target's normal capacities:

```text
target.readiness <= target.max_readiness
target.ammo <= target.max_ammo
```

Any Readiness or Ammo contribution that the target cannot accommodate is lost unless a future `Ruleset` defines an explicit recovery or storage mechanism. No such mechanism is introduced here. Exact transfer, consolidation, and rounding calculations remain undefined.

Experience follows the part of the formation actually absorbed rather than behaving like recoverable supplies. Under both remainder policies, only experience corresponding to the absorbed source contribution joins the target. The target's resulting experience reflects its existing formation together with the source contribution actually incorporated; a small veteran contribution does not make a large novice formation fully veteran.

Under `KEEP_REMAINDER`, the surviving source retains the experience associated with its remaining composition, and its resulting rank and progress may therefore need to be recalculated. Under `DISBAND_REMAINDER`, experience associated with the discarded source remainder is lost rather than transferred to the target.

Conceptually, Merge resolution may use:

```text
rank + progress
        ↓
temporary cumulative experience representation
        ↓ contribution-aware combination
resulting experience
        ↓
resulting rank + progress
```

The cumulative representation is only a conceptual Merge-resolution technique. Rank and progress remain the canonical experience state; no separate persisted veterancy value is introduced. Exact experience weighting and recalculation remain undefined.

The target retains its posture; posture is not averaged or inherited from the source. For example:

```text
Defensive target + Maneuver source → Defensive target
Maneuver target + Defensive source → Maneuver target
```

The target also retains its runtime identity, owner, `UnitDefinition`, and position. Owner, `UnitDefinition`, and tile are already compatibility constraints. If `KEEP_REMAINDER` leaves a surviving source, that source retains its own runtime identity, posture, remaining composition, and associated state.

As a general principle, formation-level runtime state follows the target unless that state explicitly defines different Merge semantics. Future state such as suppression, detection, disruption, temporary benefits or penalties, or fortification may require individual Merge rules; these are possibilities rather than current mechanics. The model does not assume that every temporary state transfers, disappears, averages, or follows health proportionally.

Merge is a Primary Action. Attack and Merge therefore cannot normally both be performed during one activation. Merge normally ends the target `UnitInstance`'s activation.

Merge must never improve the current-turn action eligibility of either participating formation. It cannot restore spent Movement, restore a spent Primary Action, grant another activation, or duplicate other opportunities already consumed during the current turn. If `KEEP_REMAINDER` leaves the source alive, its activation state is not reset and already-consumed opportunities remain consumed. The target does not become more permissive merely because it absorbed source contribution, and `DISBAND_REMAINDER` does not recover or transfer spent action opportunities. No additional rule is established here for opportunities a surviving source had not yet consumed.

Conceptually:

> resources may be consolidated; spent action opportunities may not.

The `source → target` direction therefore has real runtime meaning rather than being cosmetic.

Beyond its use of the Primary Action and its normal ending of the target's activation, the exact action cost and sequencing of Merge remain unresolved. The future turn and action-economy model, together with the `Ruleset`, may constrain when Merge is permitted, any Movement or Readiness implications, its relationship to other operations, repeated Merges, and other eligibility conditions.

## 26. Movement

Movement range is not attack range. Attack range evaluates spatial eligibility between an attacker and a target; movement is a traversal problem through the mutable world.

Within a Unit Activation, Movement is optional and remains distinct from the unit's optional Primary Action. It is not represented as a sequence of generic action points. Completing valid Movement does not by itself end the activation, though any subsequent Primary Action remains subject to that action's sequencing rules and the `Ruleset`.

A movement path traverses a sequence of neighboring tiles through their Connections:

```text
origin
  ↓
Connection
  ↓
tile
  ↓
Connection
  ↓
tile
  ...
  ↓
destination
```

Movement should be evaluated through the relationship among the moving unit, every Connection crossed, intermediate and destination `TileState`, and current world state. A destination being within a unit's nominal spatial or movement distance does not automatically mean that the unit can reach it.

Traversal may depend on:

- the `UnitDefinition` and its movement capabilities;
- the sequence of Connections crossed;
- intermediate and destination `TileState`;
- terrain;
- infrastructure;
- blockage;
- events;
- readiness;
- posture;
- other occupants;
- Ruleset constraints;
- other contextual modifiers.

Conceptually, these remain distinct:

```text
hex distance     → topological separation between tiles
traversable path → a valid sequence through current world state
movement cost    → the cost of following that path in its context
```

A path may be valid or invalid, more or less costly, and affected by world state. Two destinations at the same hex distance may therefore have different movement costs, or one may be unreachable.

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

Coordinates do not make Connections redundant. Hex distance belongs to stable spatial topology, while traversal operates through first-class Connections whose passability, infrastructure, damage, blockage, temporary modifiers, or movement consequences may change.

The same origin and destination may retain the same topological hex distance while bridge destruction, flooding, blockage, terrain modification, event effects, or other world changes alter the available route or its cost. These examples do not establish exact mechanics.

No movement-cost formula or path-validation procedure is currently frozen.

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

`Action` in this broad state-change model is not synonymous with `Primary Action` within a Unit Activation. Movement remains a distinct activation capability, while Retaliation remains a reactive combat consequence.

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
- complete Match turn lifecycle;
- exact ordering of MatchPolities and whether the higher-level turn structure is sequential, simultaneous, or phase-based;
- initiative system, if any;
- how a MatchPolity declares or finishes its turn;
- exact activation-eligibility reset implementation and action-state representation;
- whether all eligible units must activate before a MatchPolity may end its turn;
- Ruleset exceptions for multiple activations;
- detailed sequencing rules for future Primary Actions;
- victory-condition model;
- exact ruleset boundaries;
- tile behavior exactly at sea level;
- exact canonical hex-distance formula;
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
- movement-cost model;
- traversal and path-validation rules;
- how movement range interacts with movement cost;
- whether some future mechanics use non-contiguous range areas rather than simple minimum/maximum rings;
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
- exact proportional-transfer formulas and rounding behavior during Merge;
- exact Readiness consolidation calculation under `DISBAND_REMAINDER`;
- exact experience weighting and rank/progress recalculation during Merge;
- exact treatment of future temporary runtime state during Merge;
- exact Merge action cost;
- Merge timing and sequencing within a turn;
- Ruleset restrictions on repeated Merges;
- exact cost and other Ruleset restrictions for changing Posture;
- exact `Defensive` bonuses and penalties;
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
│   ├── active MatchPolity
│   └── Unit Activations
│       ├── optional Posture change
│       ├── optional Movement
│       ├── optional Primary Action
│       └── End Activation
├── Ruleset
└── Outcome
```

Conceptually:

```text
Match
  → active MatchPolity
  → eligible UnitInstances
  → Unit Activations
  → Movement / Posture / Primary Action
  → resulting runtime state
```

Merge remains a directed interaction that consolidates source contribution into a continuing target `UnitInstance`, rather than normally producing a new third entity. A partial Merge may leave the source `UnitInstance` in play under `KEEP_REMAINDER`.

The central design principle remains:

> define explicit entities and state, let actions and events produce observable consequences, and allow the world to evolve as a result.
