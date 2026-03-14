# Entity Relationships Reference

**Last Updated**: 2026-03-13

This document defines the entity relationships in SCI v0.7.0, both architecturally (via diagrams and tables) and narratively. It cross-references three authoritative sources — the Concept Assessments, the SDD, and the implemented codebase — and notes discrepancies between them.

---

## Sources

| Source | Location | Role |
|--------|----------|------|
| **Concept Assessments** | `.archive/docs/domain-transition/concept-assessments/entity-management/entities/` | Original design specifications |
| **SDD** | `docs/sdd/features/entities/` | Feature-level design documentation |
| **Codebase (Backend)** | `src/db/schema/entity/`, `src/domain/entity/` | Implemented schema and services |
| **Codebase (Frontend)** | `client/app/src/components/forms/` | Implemented UI forms |

---

## Entity Types

SCI defines **6 core entity types** and **4 sub-entity types**:

### Core Entities
| Entity | Independence | Description |
|--------|-------------|-------------|
| **Priority** | Root (top of hierarchy) | Enduring domain of sustained organisational commitment |
| **Objective** | Child of Priority | Discrete, achievable outcome advancing a Priority |
| **Action** | Child of Requirement (grandchild of Objective) | Evidence-informed response executed by a Stakeholder |
| **Stakeholder** | Root (independent) | External person, group, or organisation whose posture affects priorities |
| **Activity** | Root (cross-cutting) | Operational engagement event linking Actions, Objectives, and Stakeholders |
| **TPF** | Entity-bound | Structured communication framework translating gaps into deployable messaging |

### Sub-Entities
| Sub-Entity | Parent | Description |
|------------|--------|-------------|
| **Requirement** | Objective | Specific condition an Objective must satisfy (1–5 per Objective) |
| **Gap** | Stakeholder | Intelligence gap (information or messaging) identified on a Stakeholder |
| **Milestone** | Activity | Discrete checkpoint within an Activity |
| **TPF Point** | TPF Framework | Information Point, Messaging Point, or Talking Point within a TPF |

---

## Relationship Diagram

```
Priority
  │
  │  1:N (priorityId FK, NOT NULL)
  ▼
Objective ─── Requirements (1:N sub-entity, 1–5 per Objective)
                  │
                  │  1:N (requirementId FK, NOT NULL)
                  │  [objectiveId also stored, denormalised]
                  ▼
Action ───────────────── Stakeholder
  │   N:1 assignment      │
  │   (stakeholderId FK,   │
  │    nullable)           │
  │                        │
  │                        ├── Gaps (1:N sub-entity)
  │                        │    ├── Organisation-level (actionId = NULL)
  │                        │    └── Action-level (actionId FK populated)
  │                        │
  └── TPF (entity-bound) ──┘
       │    entityType + entityId
       │    (Stakeholder-level or Action-level)
       │
       ├── Information Points (1:N, tier=INFORMATION) ──┐
       ├── Messaging Points (1:N, tier=MESSAGING)    ───┤ via gapId FK
       └── Talking Points (1:N, synthesised)            │
                                                        ▼
                                              Gap (sub-entity of Stakeholder)
                                                ├── org-level (actionId = NULL)
                                                └── action-level (actionId → Action)


Activity (cross-cutting, no single parent)
  │
  ├── M:M → Action      (activity_action_links)
  ├── M:M → Objective   (activity_objective_links, optional requirementId)
  ├── M:M → Stakeholder (activity_stakeholder_links)
  ├── 1:N → Milestone   (sub-entity)
  ├── 1:N → TPF Point Deployments (activity_tpf_point_deployments)
  └── N:1 → Conducting Stakeholder (conductingStakeholderId FK, nullable)
```

---

## Narrative Description

### The Strategic Hierarchy: Priority → Objective → Action

The core entity chain flows downward from **Priority** through **Objective** to **Action**.

A **Priority** is the highest-level strategic commitment. It has no parent. Each Priority can have many Objectives, but an Objective belongs to exactly one Priority (`objectives.priorityId`, NOT NULL).

An **Objective** defines a discrete outcome that advances its parent Priority. Each Objective contains 1–5 **Requirements** — sub-entity records that specify conditions the Objective must satisfy. Requirements track current state, required state, and whether they have been met.

An **Action** is a targeted response assigned to a specific Stakeholder to close the gap between a Requirement's current and required states. Every Action belongs to exactly one Requirement (`actions.requirementId`, NOT NULL), which in turn belongs to an Objective. The Objective is carried as a denormalised FK (`actions.objectiveId`, NOT NULL) for query efficiency. Multiple Actions can target the same Requirement (many-to-one).

### Stakeholder: The Terminal Entity

A **Stakeholder** is an independent, external entity (person, group, or organisation) whose posture materially affects strategic priorities. Stakeholders sit outside the Priority hierarchy — they have no parent entity and are never cascade-archived.

Actions are assigned to Stakeholders via a many-to-one FK (`actions.stakeholderId`). Each Action can be assigned to at most one Stakeholder; each Stakeholder can have many Actions assigned across multiple Objectives.

**Gaps** are sub-entities of Stakeholder, classified as either `INFORMATION_GAP` or `MESSAGING_OPPORTUNITY`. Gaps can be organisation-level (no action context) or action-specific (linked via optional `actionId` FK). Gaps feed directly into the TPF pipeline.

### Activity: The Cross-Cutting Entity

An **Activity** represents an operational engagement event — a meeting, communication, research task, or advocacy effort. Activities have no single parent; instead, they link to multiple entity types via many-to-many association tables:

- **Activity ↔ Action** — via `activity_action_links` (with desiredImpact, actualOutcome, effectiveness)
- **Activity ↔ Objective** — via `activity_objective_links` (with optional requirementId scoping)
- **Activity ↔ Stakeholder** — via `activity_stakeholder_links` (engagement tracking)

Each association record carries effectiveness tracking fields. When an Activity is linked to an Action, the system auto-cascades to also link the Action's parent Objective and assigned Stakeholder.

Activities also have a separate concept of a **conducting Stakeholder** (`conductingStakeholderId` FK) — identifying when a Stakeholder is performing the activity rather than being engaged by it.

**Milestones** are sub-entities of Activity, providing discrete checkpoints with a simple status lifecycle (pending → completed/cancelled).

### TPF: Entity-Bound Communication Framework

A **Talking Points Framework** binds to a parent entity via a polymorphic pair (`entityType` + `entityId`). In practice, TPFs are created for Stakeholders (organisation-level messaging) and Actions (action-specific messaging). A Stakeholder with 3 assigned Actions may have up to 4 TPFs: 1 Stakeholder-level + 3 Action-level.

TPFs contain a three-tier point hierarchy:
1. **Information Points** — address information gaps (what the Stakeholder doesn't know)
2. **Messaging Points** — address messaging opportunities (emotional/motivational appeals)
3. **Talking Points** — synthesised from 1–3 Information + 1–3 Messaging source points

Every point must link to a specific Gap (`gapId` FK). This creates the pipeline: Evidence → Gap → Points → Deployment in Activities.

Points are deployed into Activities via `activity_tpf_point_deployments`, which tracks whether the point was modified during deployment.

#### How Points Associate with Actions

Information and Messaging Points connect to Actions through two independent axes:

1. **TPF parent binding** (`tpf_frameworks.entityType` + `entityId`) — determines *which framework* the Point lives in. An Action-level TPF (`entityType = "action"`) means all Points within it are implicitly scoped to that Action. A Stakeholder-level TPF (`entityType = "stakeholder"`) contains Points scoped to the Stakeholder as a whole.

2. **Gap content linkage** (`tpf_points.gapId` → `gaps.actionId`) — determines *which intelligence context* the Point addresses. Each Gap can be organisation-level (`actionId` NULL) or action-specific (`actionId` populated). A Point inherits its intelligence context from its linked Gap, regardless of which TPF it lives in.

These axes are independent. A Stakeholder-level TPF can contain Points that reference action-scoped Gaps (tracing back to a specific Action through `gaps.actionId`). Conversely, an Action-level TPF's Points always address Gaps on the Action's assigned Stakeholder, but those Gaps may be either action-scoped or organisation-level.

In practice, the two paths converge for Action-level TPFs: the framework is bound to the Action, and the Points within it address Gaps identified on that Action's assigned Stakeholder within that Action's context.

---

## Link Tables

| Table | From | To | Cardinality | Payload Fields | Unique Constraint |
|-------|------|----|-------------|----------------|-------------------|
| `activity_action_links` | Activity | Action | M:M | desiredImpact, actualOutcome, effectiveness, version | (activityId, actionId) |
| `activity_objective_links` | Activity | Objective | M:M | desiredImpact, actualOutcome, effectiveness, requirementId (optional), version | (activityId, objectiveId) |
| `activity_stakeholder_links` | Activity | Stakeholder | M:M | desiredImpact, actualOutcome, effectiveness, version | (activityId, stakeholderId) |
| `activity_tpf_point_deployments` | Activity | TPF Point | M:M | wasModified, modificationNotes, deployedAt, deployedBy | (activityId, pointId) |
| `talking_point_sources` | Talking Point | TPF Point | M:M | (pure junction) | (talkingPointId, pointId) |

---

## FK Relationship Summary

### Required (NOT NULL) Foreign Keys
| Child Table | FK Column | Parent Table |
|-------------|-----------|-------------|
| objectives | priorityId | priorities |
| actions | requirementId | requirements |
| actions | objectiveId | objectives (denormalised) |
| requirements | objectiveId | objectives |
| activity_milestones | activityId | activities |
| gaps | stakeholderId | stakeholders |
| tpf_points | tpfId | tpf_frameworks |
| tpf_talking_points | tpfId | tpf_frameworks |

### Optional (Nullable) Foreign Keys
| Child Table | FK Column | Parent Table | Purpose |
|-------------|-----------|-------------|---------|
| actions | stakeholderId | stakeholders | Stakeholder assignment |
| activities | conductingStakeholderId | stakeholders | "By" vs "to" distinction |
| gaps | actionId | actions | Action-specific vs org-level gap |
| tpf_points | gapId | gaps | Gap association |
| activity_objective_links | requirementId | requirements | Requirement scoping |

---

*Last Updated: 2026-03-13*