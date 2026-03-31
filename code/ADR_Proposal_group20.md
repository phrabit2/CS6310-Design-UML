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
  - x : Integer
  - y : Integer
  + distanceTo(other : Location) : Integer
  + withinDistance(other : Location, range : Integer) : Boolean

GridNode (abstract) extends Location
  - maxDistance : Integer
  - initialCost : Real
  - maintenanceCost : Real
  - powerDemand : Real
  + checkCapacity() : Boolean
  + updateParent(parentId : String) : void

Gopher extends Location
  - gopherId : String
  - damageRadius : Integer
  - damageProbability : Real
  - movementRange : Integer
  + move() : void
  + causeDamage(target : GridNode) : void
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

The current system keeps simulation data only in memory for a single run. This means billing records, transactions, issue history, and related operational data are lost when the application stops, and there is no structured way to retain, archive, or purge older records over time.

## Decision

We will add an **archival mechanism** that persists selected simulation data to local files and applies configurable retention rules. These rules will determine when records remain active, when they become archived, and when they should be purged. This will support longer-running simulations and reduce unbounded in-memory growth.

## Consequences

This modification improves data lifecycle management and realism for long-running simulation use. The main trade-offs are extra persistence logic, additional file I/O, and the need to coordinate retention behavior across multiple parts of the system.

## Acceptance Criteria

Success means simulation data persists appropriately, retention rules correctly move or purge records over time, and the system performs these actions without crashing or corrupting stored state.

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

We will add a **centralized configuration mechanism** that allows key system parameters to be managed through a configuration file and a simple GUI settings panel. This will let users adjust core simulation settings without modifying source code and will make the system better aligned with Assignment 3's interface expectations.

## Consequences

This modification improves usability and reduces reliance on hardcoded values. It also fits well with the other selected modifications because both gopher behavior and archival rules introduce new configurable parameters. The main trade-off is that configuration becomes a cross-cutting concern and will require refactoring in multiple parts of the current system.

## Acceptance Criteria

Success means core parameters can be changed without source-code edits, configuration values are validated, and the system handles invalid or incomplete configuration input safely.

## Estimated Effort

**Difficulty: Medium.** The approach is conceptually simple, but it touches multiple classes and requires both interface and validation support.

---

# Summary

| # | Modification | Type | Assignment |
|---|---|---|---|
| ADR-001 | Destructive Gophers | Functional | Required (Client) |
| ADR-002 | Archivability | Non-Functional | Required (Client) |
| ADR-003 | Configurability | Non-Functional | Team-Selected |

These three modifications work well together. Configurability supports user control over gopher behavior and archival rules, while archivability provides persistence for the broader simulation state. Together, they expand the realism, usability, and maintainability of the existing power grid system.
