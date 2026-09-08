# Procedural Memory Graph (PMG) v0.9

## Technical Design Decisions (Current Consensus)

> v0.9 finalizes the Procedural State schema: Action Groups are dependency DAGs (order-invariant compilation), objects split into a task-level Object Registry + per-state Instances, Observable Conditions are symbolic predicates with verification externalized to the Execution Contract, Temporal Constraints decompose onto existing constructs, and geometry is organized into purpose-based layers. (v0.8 established Graph First and named the PMG Perception Encoder; v0.7 decided egocentric; v0.6 settled the graph architecture; v0.5 made recovery emerge as loops; v0.4 added Procedural Relevance as the compiler filter; v0.3 consolidated the state-based Local Execution Graph.)

## Purpose

The Procedural Memory Graph (PMG) is a hierarchical procedural memory representation compiled from multi-modal demonstrations (video and audio). The demonstrations are not the final memory; they are the source material used to construct an executable procedural representation that can later be replayed, generalized, localized, and executed by different humans or robotic embodiments.

The PMG serves as the source of truth for procedural knowledge. A separate latent representation will later be generated from the PMG for efficient retrieval and planning.

## The Fundamental Philosophy

The PMG is not trying to record what happened frame-by-frame.

It is trying to model **how the world evolves while accomplishing a procedural milestone**.

Therefore the Local Execution Graph should represent world evolution, not merely action sequences.

## Design Priority: Graph First

The graph is the core of the system.

The PMG effort is fundamentally about modeling the data and its structure. The central design objective is:

> How efficiently, in how much detail, and how meaningfully can the procedural knowledge in an expert demonstration be saved?

Algorithms are downstream of the data model. Once the graph is designed, the surrounding algorithms — extracting it from demonstrations (perception), recognizing it at runtime (localization), and executing it — can be developed against a stable representation.

Structure first. Algorithms second.

## System Architecture

The complete architecture consists of three major stages.

```
Video + Audio Demonstrations
            │
            ▼
    Perception Pipeline
            │
            ▼
 Structured Observations
            │
            ▼
       PMG Compiler
            │
            ▼
Structured Procedural Memory (PMG)
            │
            ▼
    Graph Encoder
            │
            ▼
 Hierarchical Latent Memory
```

The responsibilities of each stage are:

### Stage 1 — Perception

Responsible for extracting observations from demonstrations.

Produces information such as:

* World latents
* Objects
* Human pose
* Hand pose
* Contacts
* Geometry
* Point clouds
* Speech
* Scene understanding

No procedural reasoning happens here.

### Stage 2 — PMG Compiler

The PMG compiler is a multi-pass compiler that progressively transforms demonstrations into executable procedural memory.

Rather than constructing the final graph directly from observations, the compiler follows a **coarse → fine → hierarchical** strategy.

```
Video + Audio
      │
      ▼
Pass 1 — Provisional Global Spine
      │
      ▼
Pass 2 — Procedural Expansion (Intermediate Representation)
      │
      ▼
Pass 3 — Hierarchical Semantic Compression
      │
      ▼
Final PMG
```

**Pass 1 — Online** generates a Provisional Global Spine using a lightweight LLM. These milestones are semantic hypotheses that guide graph construction.

**Pass 2 — Offline Expansion** reconstructs the richest possible Local Execution Graph and serves as the compiler's Intermediate Representation (IR).

**Pass 3 — Hierarchical Semantic Compression** validates, renames, splits, merges and inserts milestones to produce the final hierarchical PMG.

**Compiler Principle:** Expand First. Compress Later.

### Procedural Relevance as the Compiler Filter

One concern when constructing procedural memories from demonstrations is preventing uncontrolled graph growth.

The PMG does not attempt to model every observation present in a demonstration.

Instead, the compiler is always operating within the context of a Task, Milestone, and Procedural State.

By the time the Local Execution Graph is being constructed, the compiler already knows:

* which Task is being executed,
* which Milestone is currently active,
* and which Procedural State is being expanded.

This hierarchy acts as a semantic filter over perception.

Rather than asking:

> "What happened in the scene?"

the compiler asks:

> "What observations are relevant for progressing from the current Procedural State to the next Procedural State within this Milestone?"

Only procedurally relevant information is considered for graph construction.

Relevant information includes:

* intentional actions,
* observable world changes,
* contact changes,
* object interactions,
* state verification,
* alternative procedures,
* recovery strategies,
* and any observation that influences future execution.

Observations that do not contribute to procedural execution are ignored.

Examples include:

* background movements,
* human fidgeting,
* unrelated object motion,
* camera shake,
* irrelevant conversations,
* or any environmental changes that do not influence the current procedural objective.

This contextual filtering naturally prevents graph explosion without requiring aggressive post-processing.

The compiler therefore stores procedural knowledge, not an exhaustive record of everything observed during a demonstration.

### Stage 3 — Graph Encoder

After the PMG has been constructed and optionally enriched, it is encoded into a latent procedural memory.

The latent memory is not the primary knowledge representation.

Instead it serves as:

* fast retrieval
* localization
* similarity search
* planning
* generalization

The PMG always remains the editable ground truth.

Because recovery and retries appear as loops in the Procedural State graph, the encoder operates on a general directed graph with cycles, not a DAG or a linear chain.

## Hierarchy

The current hierarchy is:

```
Task
  ↓
Milestones
  ↓
Local Execution Graph
  ↓
Procedural States
```

The Task is divided into semantic Milestones.

Each Milestone owns its own Local Execution Graph.

The Local Execution Graph explains how the world changes while accomplishing that milestone.

## Milestones

A Milestone represents a semantic procedural objective.

A milestone is not directly extracted from demonstrations. Instead, it is a semantic abstraction summarizing a completed Local Execution Graph.

During online recording the system maintains a Provisional Global Spine that provides coarse semantic guidance.

After offline reconstruction, the compiler validates and reorganizes these milestones using the complete procedural evidence.

Therefore the final milestone hierarchy emerges through semantic compression over reconstructed execution graphs.

Examples:

```
Take Pan
   ↓
Pour Water
   ↓
Place Pan on Stove
   ↓
Boil Water
   ↓
Add Tea Leaves
   ↓
Infuse
   ↓
Serve
```

Each milestone is responsible only for one procedural objective.

Everything inside that objective belongs to its Local Execution Graph.

These milestones represent meaningful task progress rather than individual actions. They define the procedural backbone of the task.

## Local Execution Graph

The Local Execution Graph is fundamentally **State-based**.

Rather than alternating between States and Actions, the graph consists of Procedural States connected by state transitions.

```
State
  ↓
State
  ↓
State
```

The graph backbone is therefore entirely composed of States.

> Every node in the Local Execution Graph represents a world state. There are no action nodes, recovery nodes, or special control nodes. Procedural execution emerges from traversing between world states using actions, observable conditions, and exit criteria attached to each state.

### Compiler Intermediate Representation (IR)

The Local Execution Graph is the compiler's Intermediate Representation (IR).

It intentionally stores the richest procedural information available before semantic compression. It serves as the procedural evidence from which stable High-Level Milestones and the final PMG hierarchy are derived.

### Why States Form the Backbone

When replaying a demonstration mentally, humans tend to remember how the world changed rather than every individual action.

For example, while desoldering:

```
Joint Cold
    ↓
Joint Molten
    ↓
Joint Cleared
    ↓
Component Loose
    ↓
Component Removed
```

rather than

```
Move Iron
    ↓
Touch Joint
    ↓
Heat
    ↓
Remove Solder
    ↓
Pull Component
```

Similarly, during tea making:

```
Water Cold
    ↓
Water Heating
    ↓
Water Boiling
```

is a much more natural representation than recording every individual action as a graph node.

The world states define the procedure.

Actions, together with autonomous world evolution, drive the transition between Procedural States. Observable Conditions verify that the world has evolved as expected, while Exit Criteria determine when the current Procedural State has completed.

## Procedural State

A Procedural State represents a stable phase of execution within a Milestone.

It is not a single frame.

Instead, it spans a meaningful segment of time.

Example:

```
State

Water Heating
```

This state may last several seconds while the world continuously evolves.

### Contents of a Procedural State

Each Procedural State owns:

```
State
├── World Representation
│     ├── Procedural Geometry (spatial relations + contact graph)
│     ├── Reference Geometry (egocentric object poses)
│     └── Dense Reconstruction (by reference, via Frame Range)
├── Object Instances (references into the task Object Registry)
├── Frame Range
├── Action Group (dependency DAG of Actions)
├── Observable Conditions (symbolic; verification lives in the Execution Contract)
├── Exit Criteria (boolean expressions over conditions → target states)
├── Statistics
└── Confidence
```

### World Representation

The World Representation captures the environment during that procedural phase.

It may contain:

* scene latent
* objects
* geometry
* spatial relations
* contact graph
* human pose
* tool pose
* point cloud
* semantic attributes

Unlike the original design, this world representation may summarize multiple frames belonging to the same procedural phase.

**Reference frame:** The PMG is egocentric. The World Representation is captured relative to the demonstrator's (or executor's) viewpoint rather than a global world frame.

### Geometry Layers (current proposal)

Geometry is organized by **purpose**, not quality:

* **Procedural Geometry** — symbolic spatial relations + contact graph. Always stored. This is what procedures reason over.
* **Reference Geometry** — egocentric object poses. Always stored. Anchors localization and execution.
* **Dense Reconstruction** — point clouds, splats, meshes. Stored in a demonstration-indexed store keyed by Frame Range, and referenced — never embedded.

The PMG remains a knowledge graph, not a 3D database. Dense geometry can always be retrieved or re-derived through the Frame Range.

### Object Representation

Objects are split between a task-level registry and per-state instances. **Identity is global; state is local.**

**Object Registry (task-level)** — one entry per physical object:

```
Object
├── id
├── class
├── intrinsic properties
├── canonical geometry reference
└── affordances
```

"Kettle-1" exists exactly once per task.

**Object Instances (per-state)** — each Procedural State references registry ids and owns only what varies in that state:

```
Object Instance
├── object_id (registry reference)
├── pose (egocentric)
├── articulation / state attributes (lid open, knob at 350°C)
├── contacts
├── visibility
└── role in this state
```

Consequences:

* graph size stays linear — heavy object data is never duplicated per state,
* object identity is explicit, which State Identity depends on ("same kettle, moved" vs "a different kettle"),
* demonstration merging becomes partly a registry-alignment problem.

Scope: the registry is per-Task for now. A cross-task concept library ("kettles in general") can layer on later without schema change.

### Frame Range

Each state references the original demonstration using a Frame Range.

Example:

```
Frames

230 – 412
```

Frame Range is the canonical temporal reference.

Duration and timestamps can always be derived from it.

## Action Groups

One of the major design changes is replacing individual Action nodes with Action Groups.

A state may contain:

* zero actions
* one action
* multiple actions — concurrent, or partially ordered by dependencies

Example:

```
Actions

• Turn Knob
• Click Lighter
```

Both actions belong to the same procedural state because together they transform the world.

The Local Execution Graph itself does not become more complex.

### Why Action Groups

Multiple actions frequently cooperate to produce one world change.

Examples:

Lighting a stove:

```
Turn Knob
    +
Click Lighter
    ↓
Flame Exists
```

Neither action alone accomplishes the procedural objective.

The world changes only after both have contributed.

### Action Group Internals: A Dependency DAG

An Action Group is a **partial-order DAG of Actions**.

```
Action Group
├── actions
│     Action: verb, actor, target, motion ref, role
├── dependency edges — labeled with enabling Observable Conditions
└── coordination constraints — start-together, maintain-A-while-doing-B
```

**Edges are dependency edges, not temporal edges.** An edge means:

> "This action cannot begin until the required conditions produced by its parent action(s) are satisfied."

It does not mean "this action happened immediately afterwards."

Example:

```
Turn Knob ──produces──▶ Gas Flowing
Ready Lighter ──produces──▶ Lighter Ready

Click Lighter ◀──requires── Gas Flowing + Lighter Ready
```

Actions without dependency constraints may execute concurrently or in any order.

Coordination constraints capture genuinely simultaneous cooperation — e.g., *maintain heat with the iron while pulling the pin* during desoldering. Timing synchronization between parallel actions lives here.

### Order-Invariant Compilation

Dependency edges align with the compiler philosophy: **the PMG stores procedural knowledge, not demonstrations.**

If three experts perform

```
Rotate → Pick → Click
```

and another performs

```
Pick → Rotate → Click
```

both compile into exactly the same Action Group — because the procedural knowledge is

> Click requires Gas Flowing and Lighter Ready

not the exact order in which those preparations happened.

### Action Schema (intentionally minimal in v0.9)

For v0.9, an Action carries: verb, actor, target, motion reference, role.

Action is the one schema element without a full schema yet. The expected eventual shape — id, verb, actor, target, tool, motion_ref, dependencies, coordination_constraints, metadata — is acknowledged but deliberately not solved now.

## Observable Conditions

Earlier discussions referred to Events or Substates.

The current understanding is that these are better viewed as **Observable Conditions**.

Observable Conditions describe evidence that the world has changed.

Examples:

```
Flame Exists
Solder Molten
Drawer Open
Grasp Established
Water Temperature Rising
Steam Visible
Water Bubbling
```

Observable Conditions are not procedural steps.

They are observations confirming the current state of the world.

### Observable Condition Schema

In the graph, an Observable Condition is purely symbolic:

```
ObservableCondition
├── id
├── description (natural language)
├── predicate (subject, relation, object/value)
└── temporal qualifier (instant | sustained ≥ t | within t)
```

The predicate target may be another object or a literal value:

```
Cup — on — Table
Temperature — > — 95°C
```

**Verification does not belong to the graph.** How a condition is checked — verification method, tolerance, evidence, confidence — lives in a separate **Execution Contract**, keyed by condition id:

```
Execution Contract (per condition id, outside the graph)
├── verification method (detector / VLM query / measurement)
├── tolerance
├── evidence (frame ranges, supporting demonstrations)
└── confidence
```

This keeps the graph perception-agnostic: the memory representation does not leak assumptions about today's perception stack. The PMG Perception Encoder later decides how conditions are verified by filling the Execution Contract.

### Relationship Between Actions and Observable Conditions

Actions are intentional operations performed by an agent.

Observable Conditions are the resulting changes in the world that can be verified through perception.

For example:

```
Actions

Turn Knob
Click Lighter
    ↓
Observable Condition

Flame Exists
```

Similarly:

```
Action

Heat Joint
    ↓
Observable Condition

Solder Molten
```

The system does not execute:

```
Action
  ↓
Event
  ↓
Action
```

Instead, it executes an Action Group and continuously observes whether the expected conditions have become true.

## Exit Criteria

Each Procedural State defines Exit Criteria.

Exit Criteria determine when the compiler or execution engine should leave the current state.

Example:

```
State

Water Heating

Exit Criteria

• Water Boiling Detected
```

Once the Exit Criteria become true:

```
Water Heating
      ↓
Water Boiling
```

The transition occurs.

A Procedural State may define multiple Exit Criteria, each leading to a different next State:

```
State
Water Heating

Exit Criteria

1. Water Boiling
        ↓
   State
   Water Boiling

2. Flame Lost
        ↓
   State
   Stove Unlit
```

The graph therefore only needs to represent:

* States
* Possible outgoing transitions (based on Observable Conditions / Exit Criteria)

Everything else emerges naturally from this topology.

### Exit Criteria, Formally

> An Exit Criterion is a boolean expression over Observable Condition ids, mapped to a target State.

```
ExitCriterion
├── expression — AND / OR / NOT over ObservableCondition ids
└── target state
```

This makes the transition system fully formal: states, symbolic conditions, boolean expressions, target states — nothing else.

Timeouts need no special mechanism: a timeout is simply an Exit Criterion whose expression uses a temporal observable (e.g., `elapsed_time ≥ 5s`), typically routing to a corrective state.

## Alternative Procedures and Recovery

The PMG is designed for future execution, not merely replaying demonstrations.

Consequently, the graph should represent all procedurally meaningful ways in which a task may evolve.

Within a Procedural State, multiple outcomes may exist:

* Canonical progression
* Legitimate alternative procedures
* Recovery strategies

These are not treated as "failures" in the traditional sense.

Instead, they represent different world evolutions that remain relevant to successfully accomplishing the procedural objective.

### Recovery as Loops, Not Nodes

Recovery is not represented as a separate graph construct. Instead, recovery naturally emerges as loops within the Procedural State graph. When an observable condition indicates that the current procedural objective cannot progress, the graph transitions to an appropriate corrective state, which eventually re-enters a previously defined Procedural State. This allows the PMG to represent retries and corrective procedures without introducing specialized recovery nodes.

For example:

```
Water Heating

        │
        │ Flame Lost
        ▼

  Stove Unlit

        │
        │ Flame Exists
        ▼

Water Heating
```

When the corrective actions in "Stove Unlit" succeed, execution naturally re-enters "Water Heating", forming a loop.

### Corrective States Are World States

Corrective states follow the same convention as every other node: they are world states, not actions. The corrective *actions* live inside the state as an Action Group:

```
State

Stove Unlit

Actions
• Turn Knob
• Click Lighter

Observable Conditions
• Flame Exists

        ↓

State

Water Heating
```

Similarly, while desoldering:

```
State
Joint Heating

├── Exit:
│   Joint Molten
│          ↓
│      Joint Molten
│
└── Exit:
    Solder Not Melting
            ↓
      Joint Not Melting
      (Actions: Apply Flux)
            ↓
       Joint Heating
```

Again, no recovery node. Just a loop.

The corrective procedure is therefore not special. It is just another world-state sequence. Every node is a world state, and every transition is procedurally meaningful. Nothing in the graph says "this is recovery" — recovery is simply the path the planner follows.

This also matches how humans think. When something goes wrong, we don't think "enter recovery mode." We think "go back to where I was and try again," or "return to the previous step."

Procedurally, recovery is simply revisiting a previously known state with additional knowledge or after performing a corrective action.

Canonical execution, alternative procedures, retries, and recovery are therefore natural properties of the graph topology rather than separate mechanisms.

### The Cost of This Simplification: State Identity

By removing special recovery constructs, the design asserts:

> The graph only contains States. Everything else is navigation between existing States.

That means the compiler must answer one question reliably:

> "Have I seen this world state before?"

When the flame is relit, the compiler has to recognize that the resulting world is not a new state — it is the existing "Water Heating" state. Otherwise the graph would fragment:

```
Water Heating (1)
      ↓
  Stove Unlit
      ↓
Water Heating (2)
      ↓
  Stove Unlit
      ↓
Water Heating (3)
```

which would destroy the graph.

The design therefore trades explicit recovery mechanisms for robust state identity. This is a good trade: state identity is a foundational problem that must be solved anyway for repeated demonstrations, graph merging, localization during execution, latent encoding, and recovery loops. One foundational problem replaces multiple invented mechanisms.

**Egocentric consequence.** Because the PMG is egocentric, two visits to the same Procedural State — within one demonstration or across demonstrations — will generally be observed from different viewpoints. State matching must therefore be viewpoint-tolerant by construction. State Identity, demonstration merging, and runtime localization collapse into the same underlying problem: recognizing a world configuration from a different vantage point.

**Planned resolution — PMG Perception Encoder.** This will be addressed by a dedicated perception system whose latent space is organized around procedural equivalence rather than visual similarity: two observations embed close together when they represent the same procedural state, regardless of viewpoint or visual appearance. The PMG Perception Encoder is deliberately deferred until after the graph design is settled, since the graph defines what the encoder must recognize.

### The PMG Is a Directed Graph with Cycles

Because recovery and retries appear as loops, the Local Execution Graph is a general directed graph with cycles, not a DAG:

```
        ┌─────────────┐
        │             │
        ▼             │

State A → State B → State C
   ▲                  │
   └──────────────────┘
```

The Graph Encoder must assume this topology. This is closer to real procedural memory: humans retry, recover, and revisit earlier procedural phases.

Only procedurally meaningful alternatives are represented.

Incidental deviations such as accidental hand movements, unrelated environmental changes, or demonstrator-specific behaviour are intentionally excluded.

The Milestone and Procedural State hierarchy naturally constrains which alternative paths are considered relevant.

As additional demonstrations are processed, the PMG accumulates canonical procedures, legitimate alternatives, and recovery loops into a unified procedural memory.

The graph therefore captures how experts successfully accomplish tasks under varying conditions, rather than only recording an idealized linear execution.

## State Changes Without Actions

One important realization is that not every state transition requires an action.

Example:

```
Water Heating
      ↓
Water Boiling
```

No additional human action is required.

Physics alone causes the world to evolve.

The PMG naturally supports this because Action Groups are optional.

## Example — Boil Water

Inside the Milestone:

```
Boil Water
```

the Local Execution Graph may look like:

```
State

Water Cold

Actions
• Turn Knob
• Click Lighter

Observable Conditions
• Flame Exists

Exit Criteria
• Stable Flame

        ↓

State

Water Heating

Actions
None

Observable Conditions
• Temperature Rising
• Steam Appearing

Exit Criteria
• Rolling Boil Detected

        ↓

State

Water Boiling

Actions
None

Observable Conditions
• Continuous Bubbling
```

This representation closely follows how the world actually evolves.

## Example — Desoldering

Milestone:

```
Heat Joint
```

Local Execution Graph:

```
State

Joint Cold

Actions
• Position Iron
• Apply Heat

Observable Conditions
• Solder Melting

Exit Criteria
• Joint Fully Molten

        ↓

State

Joint Molten

Actions
• Use Solder Wick

Observable Conditions
• Solder Removed

Exit Criteria
• Pad Cleared

        ↓

State

Joint Cleared
```

Again, the graph follows the evolution of the world rather than individual manipulations.

## Event-Driven Graph

The PMG is fundamentally event-driven.

New nodes are created only when meaningful procedural events occur.

Time passing alone never creates new graph nodes.

For example:

```
Cup on table

(wait 10 seconds)

Cup on table
```

does not create another State.

Instead, temporal information is attached to the existing State.

## Temporal Information

Frame Range is the canonical temporal reference.

Every State references the original demonstration through:

* Frame Range

From this information:

* timestamps
* duration
* temporal position

can be derived when required.

Explicit storage of entry time, exit time, and duration is therefore unnecessary unless future applications require it.

## Temporal Constraints

Temporal Constraints are different from observed timing.

Observed timing describes what happened in a demonstration.

Temporal Constraints describe how execution should occur.

They are procedural execution constraints rather than recorded observations — but they are **not a separate construct**. Every temporal constraint decomposes onto the existing schema:

| Constraint | Where it lives |
|---|---|
| "Hold for at least 2 seconds" | Temporal qualifier on an Observable Condition (`sustained ≥ 2s`) |
| "Finish within 5 seconds" | Timeout Exit Criterion — a timeout is just another transition |
| "Wait until water boils" | Ordinary Exit Criterion over an Observable Condition |
| "Maintain contact before lifting" | Dependency edge / coordination constraint inside the Action Group |
| "Synchronize with another action" | Coordination constraint inside the Action Group |
| "Wait 10 seconds" | Action-free State whose Exit Criterion is `elapsed_time ≥ 10s` |

**Time is just another observable.** `elapsed_time` is an Observable Condition like any other, which is why a pure waiting phase is simply an action-free Procedural State — fully consistent with physics-driven state evolution.

## Procedural Memory Philosophy

The PMG is not intended to be a recording of demonstrations.

Instead, it is a compiled procedural representation.

The compiler transforms demonstrations into executable procedural knowledge.

Demonstrations are therefore considered training data rather than the memory itself.

## PMG as Ground Truth

The PMG is always considered the authoritative procedural memory.

Future systems may enrich the PMG with additional knowledge, including:

* expert annotations
* execution statistics
* robot experience
* inferred affordances
* learned policies
* VLM reasoning

These enrichments modify or extend the PMG.

The latent memory is regenerated from the updated graph rather than edited directly.

## Hierarchical Latent Memory

Once the PMG is finalized, multiple procedural abstraction levels can be encoded.

Potential embedding levels include:

* State embeddings
* Action Group embeddings
* Local execution graph embeddings
* Milestone embeddings
* Task embeddings

This enables retrieval at different semantic resolutions while preserving the explicit symbolic graph.

## Design Philosophy

The PMG follows a hybrid symbolic-neural architecture.

Symbolic Graph:

* interpretable
* editable
* verifiable
* mergeable
* executable

Latent Memory:

* searchable
* compressible
* generalizable
* localizable
* efficient for planning

The graph represents knowledge.

The latent representation represents efficient access to that knowledge.

### Knowledge vs. Execution Contracts

> The Procedural State schema intentionally separates procedural knowledge from execution mechanisms. The graph stores symbolic procedural knowledge, while execution-specific concepts such as perception verification, motion generation, controller implementation, and scheduling remain external contracts that operate over the graph rather than becoming part of it.

This is the recurring design principle behind the v0.9 schema decisions: Observable Conditions are symbolic while verification lives in the Execution Contract; Actions carry motion *references* rather than controllers; and the Perception Encoder operates over the graph rather than inside it.

### Local Execution Graph Philosophy

The Local Execution Graph should represent world evolution inside a procedural milestone.

* Every node is a world state — there are no action, recovery, or control nodes.
* The graph backbone is composed entirely of Procedural States.
* Actions are execution mechanisms.
* Observable Conditions are perceptual evidence.
* Exit Criteria determine when the current procedural phase has completed.

This representation naturally supports:

* concurrent and dependency-ordered actions,
* action-free state evolution driven by physics,
* perception-based execution verification,
* compact procedural memory,
* and an egocentric representation that remains faithful to real demonstrations while being directly executable by humans or robots.

## Open Design Questions

The graph architecture is now largely settled. The remaining work is not about inventing new structures — it is about defining the compiler algorithms that decide what constitutes a state and how states are recognized across time and demonstrations. Those algorithms are likely to be the core technical contribution of the PMG system.

### Highest Priority

Two questions now dominate the project. Their answers will determine recovery loops, demonstration merging, graph size, latent embeddings, and execution robustness.

1. **State Identity** — When are two observed world configurations considered the same Procedural State? This governs loops, localization, and graph compression. *Resolution path: the PMG Perception Encoder — a latent space organized around procedural equivalence rather than visual similarity — to be designed after the graph is settled.*
2. **State Segmentation** — When should the compiler emit a new Procedural State instead of extending the current one? This defines the boundaries of the Local Execution Graph. (Absorbs the earlier questions "What constitutes a meaningful new State?" and "How do we define the start and end boundaries of a State?")

### Remaining Questions

3. How much geometric information should each State own? *Leading proposal (v0.9): purpose-based Geometry Layers — Procedural Geometry and Reference Geometry stored in-state, Dense Reconstruction by Frame Range reference.*
4. What is the primary optimization objective of the PMG?
5. How should repeated demonstrations merge into a single graph?
6. How should uncertainty be represented? *Leading proposal (v0.9): a uniform metadata block {confidence, provenance, support} attachable to any schema element, plus per-transition traversal statistics. Deliberately designed last — pure metadata layered onto the schema.*
7. What is the runtime localization strategy?
8. How should reusable procedural skills be represented?
9. What hierarchy beyond Milestones should exist?
10. What is the full Action schema? *Intentionally minimal in v0.9 (verb, actor, target, motion ref, role); expected eventual shape acknowledged in the Action Schema section.*

### Resolved

* ~~Should the PMG be world-centric, egocentric, or hybrid?~~ **Decided: egocentric.** The World Representation is captured relative to the demonstrator's (or executor's) viewpoint. (v0.7)
* ~~How should the internal structure of Action Groups be represented?~~ **Decided: a partial-order dependency DAG.** Edges are dependency edges labeled with enabling Observable Conditions; actions without dependencies may run concurrently or in any order. Compilation is order-invariant across demonstrations. (v0.9)
* ~~Where should timing synchronization between parallel actions live?~~ **Decided: coordination constraints inside the Action Group**, with duration/timeout semantics expressed as temporal qualifiers on Observable Conditions and Exit Criteria. (v0.9)
* ~~Should objects exist independently of States?~~ **Decided: yes.** Identity lives in a task-level Object Registry; States own per-state Object Instances that reference it. (v0.9)
* ~~Where do Temporal Constraints live?~~ **Decided: decomposed onto existing constructs** — temporal qualifiers, timeout exits, coordination constraints, and elapsed_time as an Observable Condition. No separate temporal construct exists. (v0.9)
