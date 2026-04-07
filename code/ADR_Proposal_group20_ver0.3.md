# CS6310 – Software Architecture & Design
# Assignment #3: Phase 3 — ADR Proposals
## Group 20 | Spring 2026

---

# ADR-001: Destructive Gophers (Functional Modification)

**Type:** Functional Modification  
**Assignment:** Required (Client-Designated)  
**Date:** 2026-03-23  
**Status:** Proposed

## Context

The current power grid simulation is mostly static because infrastructure continues operating unless a user manually changes it. As a result, the system cannot currently represent environmental hazards that damage infrastructure and disrupt customer service. The existing class model already includes location and distance concepts through `GridNode`, which provides a natural basis for a proximity-based hazard.

However, `GridNode` currently combines two concerns: **spatial positioning** (`x`, `y`, `distanceTo()`, `withinDistance()`, `maxDistance`) and **power infrastructure behavior** (`initialCost`, `maintenanceCost`, `powerDemand`, `checkCapacity()`, `updateParent()`). Introducing a non-infrastructure entity such as a gopher directly under `GridNode` would force it to inherit attributes and operations that have no meaningful application to a destructive agent.

## Decision

We will introduce a new abstract class **`Location`** that encapsulates spatial positioning and distance calculation, and restructure `GridNode` to inherit from it. A new **`Gopher`** class will also inherit from `Location`, giving it coordinate-based positioning and proximity detection without inheriting power infrastructure logic.

```
Location (abstract)
  # x : Integer
  # y : Integer
  + distanceTo(other : Location) : Integer
  + withinDistance(other : Location, range : Integer) : Boolean

GridNode (abstract) extends Location
  # maxDistance : Integer
  # initialCost : Real
  # maintenanceCost : Real
  # powerDemand : Real
  + checkCapacity() : Boolean
  + updateParent(parentId : String) : void

Gopher extends Location
  - gopherId : String
  - damageRadius : Real
  - damageProbability : Real
  - damageLevel : Real
  - movementRange : Integer
  - movementMode : MovementMode
  + moveTo(newX : Integer, newY : Integer) : void
  + moveRandomly() : void
  + applyDamage(target : GridNode) : void

enum MovementMode {
  CONTROLLED
  RANDOM
}
```

When a gopher is near a relevant power infrastructure element, it will have a configurable chance of causing partial or total damage based on proximity. This will let the simulation represent outages and reduced service caused by nearby hazards.

## Consequences

This modification improves realism by introducing dynamic failures that affect supply, maintenance, and customer impact. Extracting `Location` as a separate abstraction cleanly separates spatial concerns from infrastructure concerns, and ensures `Gopher` does not inherit inapplicable infrastructure attributes. It also builds naturally on the current class structure instead of requiring a completely new model. The main trade-offs are added implementation complexity in failure propagation and more difficult testing when random behavior is enabled.

## Acceptance Criteria

Success means users can create and move gophers, observe damage to nearby infrastructure, and see the effect on service delivery. The system must support the proximity, movement, and damage behavior described in the Assignment 3 modification.

## Estimated Effort

**Difficulty: Medium-High.** This feature affects both the infrastructure model and downstream system behavior, so it will require moderate to significant implementation effort.

---

# ADR-002: Archivability (Non-Functional Modification)

**Type:** Non-Functional Modification  
**Assignment:** Required (Client-Designated)  
**Date:** 2026-03-23  
**Status:** Proposed

## Context

The current system keeps simulation data only in memory for a single run. As the simulation progresses, historical business records (such as bills and payments) accumulate indefinitely. There is no structured way to retain, archive, or purge older financial and operational records over time, leading to unbounded memory growth.

_Note: The actual state of the simulation grid itself is handled separately by the ConfigurationManager (see ADR-003)._

## Decision

We will add an **archival mechanism** that persists selected simulation data to local files and applies configurable retention rules. These rules will determine when records remain active, when they become archived, and when they should be purged. This will support longer-running simulations and reduce unbounded in-memory growth.

## Consequences

This modification improves data lifecycle management and realism for long-running simulation use. The main trade-offs are extra persistence logic, additional file I/O, and the need to coordinate retention behavior across multiple parts of the system.

## Acceptance Criteria

Success means business data (Bill, Transaction, EquipmentIssue) persists appropriately, retention rules correctly move or purge records over time, and the system performs these actions without crashing or corrupting stored state.

## Estimated Effort

**Difficulty: Medium.** The core idea is straightforward, but it affects several existing data areas and requires careful persistence handling.

---

# ADR-003: Configurability (Non-Functional Modification)

**Type:** Non-Functional Modification  
**Assignment:** Team-Selected (Chosen)  
**Date:** 2026-03-23  
**Status:** Proposed

## Context

The current simulation relies on several hardcoded values, such as billing settings, grid distance limits, and capacity constraints. As the system grows, especially with the addition of gopher behavior and archival rules, this makes the application harder for users to control and harder for developers to maintain.

## Decision

We will introduce a **centralized ConfigurationManager** designed to serve as the system’s primary state-management engine. Rather than focusing solely on static settings, this component will incorporate a dual-purpose architecture:

**Parameter Management:** It will provide a centralized mechanism for managing simulation constants—such as billing rates and distance constraints—through JSON configuration files and the GUI settings panel.

**State Persistence:** It will function as a "state-saving machine" responsible for serializing and deserializing the active simulation environment. This includes the live grid topology (parent/child relationships), the real-time health levels of infrastructure, and the current positions of all Gophers.

This integrated approach ensures that when a user saves or loads a simulation, both the physical "board" and the specific rules governing that run are preserved in a single, consistent operation.

## Consequences

This modification improves usability and reduces reliance on hardcoded values. It also fits well with the other selected modifications because both gopher behavior and archival rules introduce new configurable parameters. The main trade-off is that configuration becomes a cross-cutting concern and will require refactoring in multiple parts of the current system.

## Acceptance Criteria

Success means core parameters can be changed without source-code edits, configuration values are validated, and the system handles invalid or incomplete configuration input safely.

## Estimated Effort

**Difficulty: Medium.** The approach is conceptually simple, but it touches multiple classes and requires both interface and validation support.

---

# ADR-004: Simulation Manager (Architectural Addition)

**Type:** Architectural Core Update
**Date:** 2026-04-01
**Assignment:** Required (Client constraints regarding system time)
**Status:** Proposed

## Context
The assignment documentation explicitly states that the simulation is not allowed to use live system clocks or wait on real-world time. Because environmental hazards (Gophers), billing cycles, and power consumption must advance logically, the application requires a deterministic way to move "time" forward without relying on Java thread sleeps or system timers.

## Decision
We will introduce a central SimulationManager. This class will act as the core engine of the system, managing a logical currentTick integer. All time-based events—such as gopher movement, grid capacity checks, and billing cycle transitions—will be triggered sequentially when the manager advances the tick.

## Behavior
Manual Advancement: The GUI will provide a "Step" or "Advance Time" button. Clicking this calls processSingleTick().

Event Orchestration: Inside processSingleTick(), the manager will sequentially command Gophers to move/damage, ask the Grid to supply power, and update the PowerCompany billing cycles based strictly on the logical tick.

Time Travel: The jumpToTick(targetTick) method will allow the simulation to load a past state by coordinating with the ArchiveManager and ConfigurationManager to overwrite the current state with historical data.

## Acceptance Criteria
Success means the simulation progresses logically only when a tick is processed. Gopher movements, damage events, and billing updates must be reproducible identically given the same sequence of ticks and inputs.

## Estimated Effort
Difficulty: Medium/Hard. The logic is simple, but wiring all existing components (PowerCompany, GridNodes, Gophers) to respond to the processSingleTick() loop will require structural refactoring.

# Summary

| # | Modification | Type | Assignment |
|---|---|---|---|
| ADR-001 | Destructive Gophers | Functional | Required (Client) |
| ADR-002 | Archivability | Non-Functional | Required (Client) |
| ADR-003 | Configurability | Non-Functional | Team-Selected |
| ADR-004 | Simulation Manager | Core Architecture | Required (Constraints) |

These four modifications work well together. Configurability supports user control over gopher behavior and archival rules, while archivability provides persistence for the broader simulation state. Together, they expand the realism, usability, and maintainability of the existing power grid system.
