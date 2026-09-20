# Aces — Match Lifecycle

**Status:** Exploratory  
**Last reviewed:** 2026-09-20<br>
**Responsibility:** Define the current temporal gameplay lifecycle within a Match, from Round start through Round completion.  
**Canonical for:** Detailed lifecycle phases, TurnOpportunity semantics, activation sequencing, and spent-opportunity invariants.  
**Depends on:** [Match](overview.md#9-match), [ownership and control](overview.md#8-ownership-vs-control), [Ruleset](overview.md#10-ruleset), [Readiness and Unit Posture](overview.md#22-readiness-and-unit-posture), [Unit Merge](overview.md#unit-merge), [Movement](overview.md#26-movement), and [Events](overview.md#28-events).  
**Parent overview:** [Game Design Overview](overview.md)

`Last reviewed` means the date the document's conceptual content was last reviewed or materially updated, not necessarily the date of the Git commit.

Within this document's declared scope, canonical means authoritative for the current evolving design; it does not mean frozen or final. Exploratory means that the current design may evolve, not that its current agreed design is non-authoritative.

## Scope

This document owns the detailed lifecycle design for Round, World Resolution as a lifecycle phase, `TurnOpportunity`, `MatchPolity` turns, Unit Activation opportunities, activation-level sequencing, End Activation, spent-opportunity invariants, and Round completion.

It does not own the broader Match configuration and status lifecycle, detailed Unit state, combat formulas, detailed Posture mechanics, detailed Merge resource semantics, Movement or pathfinding mechanics, Events as a complete domain, or economy or resources. Those concepts remain in the [Game Design Overview](overview.md) and may be referenced here only where they interact with lifecycle timing or action eligibility. This document records initiative and ordering algorithms as unresolved lifecycle questions; it does not define or imply those algorithms.

## Round, Turn Opportunities, and Unit Activation

The original brainstorm proposed turns by republic.

Under the current model, a `Round` is the global temporal cycle within a `Match`, above any individual `MatchPolity` turn. It is systemic Match time, not a player-controlled action or a Unit Activation.

Conceptually:

```text
Match
└── Round
    ├── World Resolution
    ├── Turn Opportunities / MatchPolity Turns
    └── Round completion
```

A Round normally represents a cycle in which the MatchPolities eligible when its normal opportunities are established receive their corresponding Turn Opportunities. The initial model is sequential, but neither initiative nor the exact ordering or selection of MatchPolities is defined yet.

### World Resolution

World Resolution occurs once at the beginning of each Round, before normal `TurnOpportunity` instances are established. It processes global or world-scoped recurring state and Events, then finishes with a resolved World state from which the Round takes its normal Turn Opportunity snapshot.

Its conceptual order is:

```text
current World
  → advance existing world state and active Events
  → resolve expirations and transitions
  → evaluate new Event triggers
  → resolve resulting consequences and interactions
  → stabilize World
  → establish normal Turn Opportunities
```

Existing persistent state and active Events are processed before new Event triggers are evaluated. An active Event may evolve to another state or phase without ending and being triggered again. New triggers evaluate against the World produced by prior progression, transitions, and expirations, and their consequences may be applied during the same World Resolution before Turn Opportunities are established.

This is conceptual lifecycle ordering. It does not determine whether Events are technically processed serially, in batches, through dependency relationships, with queues or transactions, or by another mechanism. It also does not define probabilities, durations, formulas, weather simulation, generic Event priorities, conflict algorithms, or hidden implementation ordering.

World Resolution does not process `MatchPolity`-specific maintenance. That remains a responsibility of Begin Turn Resolution.

#### Event State and Consequences During World Resolution

The following Event concepts are defined here only as far as World Resolution requires them. They do not constitute the complete Events-domain design.

Conceptually, an `EventDefinition` supplies the Ruleset-defined identity and rules for an `EventInstance`. An `EventInstance` exists at runtime and may maintain enough state to represent progression across multiple Rounds, including phases or states that transition during World Resolution. Persistence and multiple phases are not required for every Event: an instantaneous Event may begin, produce consequences, and finish within one World Resolution.

An Event represents what is occurring or occurred. Its consequences represent the changes or modifiers it produces in the World or its entities. Ending an `EventInstance` does not by itself revert every consequence. A temporary consequence tied to the active Event may disappear when it ends, while a persistent World mutation may remain.

Events and their consequences may concern tiles, regions, Connections, Structures, or other relevant World state without requiring a separate Event concept for each target type. The `Ruleset` determines which Events may exist and how they evolve and produce consequences, but the technical representation of those rules remains undefined. The model does not yet define generic Event chains, though it does not prevent one Event from later enabling or causing another when explicit rules are introduced.

#### Event Interaction and World Stabilization

Events may interact when their scopes or consequences affect overlapping World state. Any relevant interaction or conflict must follow explicit domain or `Ruleset` semantics rather than accidental iteration or implementation order. Rules may define coexistence, exclusion, replacement, combination, or priority where a specific interaction needs it; this does not establish a generic Event-priority system.

A consequence produced during World Resolution may affect the evaluation of a later consequence when an explicit rule requires that relationship. Neither universal independence nor universal sequential execution is assumed. An Event or consequence may also invalidate another pending Event when the `Ruleset` explicitly defines that result.

World Resolution ends only after all processing belonging to that World Resolution has produced a resolved, stable World state, which becomes the observable basis for the rest of the Round. Stable means that the work assigned to the current resolution has been resolved; it does not mean recursively executing every possible future consequence until an absolute equilibrium is reached. Consequences that explicitly belong to a later Round remain pending or persistent and are not pulled into the current Round merely because they could be calculated.

Given the same relevant World state, active `EventInstances`, `Ruleset`, and already-determined random outcomes, World Resolution should produce the same result. Hidden implementation ordering must not change the gameplay outcome. The normal Turn Opportunity snapshot is established only after this stabilization; eligibility changes after the snapshot continue to follow the Turn Opportunity semantics below.

Weather is conceptually a World-scoped Event and should normally evolve once per Round rather than once per `MatchPolity` turn. This prevents adding MatchPolities from artificially accelerating global world evolution. Technical World Resolution processing, weather probabilities, climate simulation, and Event algorithms remain undefined.

World Resolution owns the lifecycle placement of that processing, not the complete [Events](overview.md#28-events) domain.

### Turn Opportunities and Round Snapshot

After World Resolution, the Round establishes its normal `TurnOpportunity` instances as a logical snapshot. Only MatchPolities eligible at that point receive a normal Turn Opportunity. The existence of a `MatchPolity` and its eligibility for a Turn Opportunity are distinct concepts; the exact eligibility and elimination conditions remain open.

A Turn Opportunity belongs to a `MatchPolity`, not to a `User` or one of the Polity's controlled entities. Ownership or control changes do not create new Turn Opportunities. The Turn Opportunity is also distinct from the activation state of any `UnitInstance`.

Its normal lifecycle is:

```text
pending → active → completed
```

A pending opportunity may instead be resolved through cancellation:

```text
pending → cancelled
```

Both `completed` and `cancelled` are terminal states for that Round. Under the initial sequential model, only one Turn Opportunity is active at a time. This does not prohibit a future `Ruleset` from deliberately using a different higher-level turn model.

If a MatchPolity with a pending Turn Opportunity becomes ineligible before its turn, that opportunity becomes `cancelled`. If a new MatchPolity appears or becomes eligible after the normal opportunities have been established, it does not automatically receive a normal Turn Opportunity in the current Round. It may receive one in the following Round if eligible when that Round's snapshot is established. This keeps normal Round membership stable and prevents implicit extra turns.

A future `Ruleset` may deliberately create an extraordinary Turn Opportunity during an active Round, but its mechanics remain undefined.

### MatchPolity Turn Lifecycle

Once a pending Turn Opportunity is selected, it becomes active and its `MatchPolity` turn follows this conceptual lifecycle:

```text
pending Turn Opportunity
  → active
  → Begin Turn Resolution
  → Activation Window
  → End Turn Resolution
  → completed
```

The turn belongs conceptually to a `MatchPolity`, not to a `User`, `Team`, or `UnitInstance`.

```text
TurnOwner = MatchPolity
```

Users operate entities through the participation, permissions, and control relationships already described. User identity does not own the game turn, and changing control between users does not create a new turn, Turn Opportunity, or refreshed Unit Activation. This separation supports human-controlled and NPC-controlled MatchPolities as well as future delegated or shared control models.

### Begin Turn Resolution

Begin Turn Resolution occurs before determining which `UnitInstances` may activate. It processes MatchPolity- or entity-specific recurring state, including established conceptual responsibilities such as:

- passive Readiness consumption;
- Posture-dependent passive Readiness behavior;
- recurring consequences of `Readiness = 0`;
- other future `Ruleset`-defined recurring effects.

These responsibilities do not imply additional maintenance mechanics. Relevant state is stabilized before the player acts. A unit destroyed or rendered ineligible during Begin Turn Resolution does not subsequently receive an activation opportunity for that turn.

The detailed meanings and effects of Readiness and Posture remain in [Readiness and Unit Posture](overview.md#22-readiness-and-unit-posture).

### Activation Window and End Turn Resolution

After Begin Turn Resolution stabilizes relevant state, the game determines eligible `UnitInstances` and opens the Activation Window. During that window, the MatchPolity may use Unit Activations according to the semantics below.

A MatchPolity is not required to activate every eligible UnitInstance before ending its turn. It may voluntarily finish the turn; unused Unit Activation opportunities then expire for that MatchPolity turn and do not carry into the next Round.

End Turn Resolution exists conceptually. For now, its minimum responsibilities are to close the Activation Window, finalize strictly turn-scoped state, and complete the active Turn Opportunity. Additional End Turn effects remain undefined.

### Round Completion

After a MatchPolity turn completes, its active Turn Opportunity becomes `completed`. If pending opportunities remain, another pending Turn Opportunity may be selected. If no pending or active opportunities remain, the Round may complete; both completed and cancelled opportunities count as resolved.

A new Round establishes its own Turn Opportunities. A MatchPolity does not retain a permanent “has acted” state merely because it acted in a previous Round.

### Unit Activation

Within an active MatchPolity turn:

```text
Activation Window
  ↓
eligible UnitInstances
  ↓
Unit Activations
```

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

Lifecycle transitions do not themselves reset Posture. The current persistence and operational rules for Posture remain in [Readiness and Unit Posture](overview.md#22-readiness-and-unit-posture).

### Movement and Primary Action

Movement and Primary Action are distinct, optional parts of a Unit Activation. Movement represents traversal of a valid path and may cross multiple neighboring tiles and Connections. It remains governed by movement capability, Readiness, Posture, world state, Connections, movement cost, and `Ruleset` constraints; it is not a sequence of generic action points.

A Primary Action is the unit's principal tactical action and may normally be consumed at most once per activation. The currently known Primary Actions are Attack and Merge. Other Primary Actions may be introduced later, but are not current mechanics.

A unit may voluntarily end its activation without using its available Movement, Primary Action, or Posture change opportunity.

Detailed path traversal and movement cost remain in [Movement](overview.md#26-movement). Detailed Posture behavior remains in [Readiness and Unit Posture](overview.md#22-readiness-and-unit-posture).

### Sequencing and End Activation

Movement does not automatically end an activation. After completing valid Movement, a unit may retain its Primary Action when that specific action and the `Ruleset` permit it. Each Primary Action may define its own sequencing restrictions; this does not establish a universal action-sequencing system.

The default Attack sequence permits Movement followed by Attack when all movement, targeting, combat, and `Ruleset` constraints are satisfied:

```text
Movement → Attack → End Activation
```

Attack followed by Movement is not permitted by default. Attack normally ends the Unit Activation, so a unit that attacks before moving does not then gain a movement opportunity.

Merge is also a Primary Action, so Attack and Merge cannot normally both occur during one activation. Merge normally ends the target `UnitInstance`'s activation. Its formation and resource semantics remain in [Unit Merge](overview.md#unit-merge).

Once a Primary Action has ended an activation, the unit cannot subsequently move, change Posture, perform another Primary Action, or voluntarily reopen that activation. This does not require every possible future Primary Action to end activation; future actions may define different semantics.

A unit may also explicitly end its activation and forgo any unused opportunities. Once completed, the activation cannot normally be reopened, and the unit cannot normally activate again during the same `MatchPolity` turn. Only an explicit future `Ruleset` rule may override this.

More generally, no operation should improve current-turn action eligibility merely by transforming, merging, transferring, or otherwise changing a `UnitInstance`:

> state or resource changes do not restore spent action opportunities.

Merge must never improve the current-turn action eligibility of either participating formation. It cannot restore spent Movement, restore a spent Primary Action, grant another activation, or duplicate other opportunities already consumed during the current turn. If `KEEP_REMAINDER` leaves the source alive, its activation state is not reset and already-consumed opportunities remain consumed. The target does not become more permissive merely because it absorbed source contribution, and `DISBAND_REMAINDER` does not recover or transfer spent action opportunities. No additional rule is established here for opportunities a surviving source had not yet consumed.

Conceptually:

> resources may be consolidated; spent action opportunities may not.

Beyond its use of the Primary Action and its normal ending of the target's activation, the exact action cost and sequencing of Merge remain unresolved. The future turn and action-economy model, together with the `Ruleset`, may constrain when Merge is permitted, any Movement or Readiness implications, its relationship to other operations, repeated Merges, and other eligibility conditions.

## Current Open Questions

The following lifecycle areas remain intentionally unresolved:

- technical World Resolution processing mechanisms, including serial, batch, dependency-based, queue-based, transactional, or other implementation models;
- exact ordering of MatchPolities within a Round;
- fixed versus calculated MatchPolity order;
- exact selection algorithm for a pending Turn Opportunity;
- detailed simultaneous or phase-based alternatives to the initial sequential model;
- initiative system, if any;
- exact MatchPolity eligibility and elimination rules;
- extraordinary Turn Opportunity mechanics;
- exact activation-eligibility reset implementation and action-state representation;
- Ruleset exceptions for multiple activations;
- detailed sequencing rules for future Primary Actions;
- additional End Turn Resolution effects;
- exact Merge action cost;
- Merge timing and sequencing within a turn;
- Ruleset restrictions on repeated Merges.

These questions should be resolved progressively through game-design iteration rather than prematurely frozen.
