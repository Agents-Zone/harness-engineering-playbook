# Evolution: Continuous Iteration of Specs and Verification

The introduction defined two principles for harness: closed-loop control and continuous evolution. The previous two chapters established the closed loop, using specs to define intent and verification to confirm execution. But we have not yet addressed the second principle.

Code does not reach the end of its life the moment it is completed. That moment is the beginning. After launch, user feedback arrives, product requirements change, features are added and modified. Your spec and tests worked well for the first version. But what happens when the second requirement comes in?

The intuitive response takes two forms, both problematic. The first is to fall back to Vibe Coding: talk to the Agent about the new requirement directly, skipping the spec. The previous two chapters already explained why this does not work. Attention decay, intent loss, verification losing its anchor point: all the old problems resurface.

The second is to manually update all documentation: reflect the new requirement's impact across the spec, tests, and project-context one by one. The direction is correct, but if the spec was done thoroughly, the document volume is substantial. Features are intertwined. Changing one business rule can affect tests in three modules and constraints in two specs. The cost of manual maintenance grows rapidly with project scale.

This chapter addresses the second principle of harness: how to make the closed loop evolve alongside the project, instead of beginning to rot after the first iteration.

## Why Specs Inevitably Go Stale

Spec staleness is not negligence. It is structurally inevitable. The closed loop's frame of reference is the spec. All verification actions are anchored to it. If the spec is correct, the closed loop guarantees correct output. But the spec does not update itself.

The introduction already pointed this out when discussing the Agent's lack of accumulated memory: the external knowledge system you build for the Agent (specs, tests, process documents) substitutes for the business intuition and project memory that humans carry naturally. But knowledge in the human mind updates automatically as the project evolves. Documents do not. You refactor a module, and your understanding of that module updates in sync. The documentation stays where it was before the refactoring.

Every iteration creates drift between spec and code. A new feature changes system behavior, but the spec does not know. Tests for old features still verify premises that no longer exist. After several rounds, the distance between spec and code grows, and the closed loop is not losing its protection. It is protecting the wrong target. This is exactly how the scenario warned about in the introduction occurs: deviation becomes institutionalized under the closed loop's protection.

Keeping documentation in sync with code is not a problem invented in the AI era. Traditional software engineering faced it decades ago and developed mature methodologies. But can those methodologies be applied directly?

## Agile Has the Right Principle, Wrong Granularity

Agile methodology's iteration principles apply directly to spec maintenance: small increments, continuous feedback, embrace change. But agile's execution granularity was calibrated for human teams. Applying it directly to AI development produces a severe efficiency inversion.

The BMAD framework is currently the most complete agile transplant in the AI development community. It decomposes epics into stories, equips each story with five or more documents, assigns twelve agent personas to collaborate, and routes each story through a full approval gate. This process ensures every step is documented and reviewed. But a single story corresponds to a few days of work for a human developer. Equipping that size of work with a full process is cost-justified in human teams.

An Agent's execution unit is far larger. The context capacity concept established in the specification chapter explains this mismatch: the amount of information a single Agent session can effectively process covers the workload of multiple traditional stories. In our example project AILock-Step, a single feature includes a complete spec, task list, checklist, implementation, and verification, equivalent in scale to multiple BMAD stories. Running each story-level unit through a full agile process means the human spends more effort on process management than the Agent saves in execution time. In practice, this efficiency inversion is obvious: a requirement that can be described in a few sentences may take an entire day to process through the full agile workflow.

The root cause is that the execution unit changed in size, but the process granularity did not change with it. Agile's iteration management principles still apply. What needs recalibration is the size of each step.

## Manage Docs Like You Manage Code

Iterate by feature. Each feature runs the full spec, implement, verify closed loop internally. The remaining question is: how do you manage documentation across features? Each iteration modifies the same set of spec files. You need to trace modification history and detect inconsistencies. These challenges are structurally identical to code management challenges. Code gets tested after merging; specs need verification after merging too. Code has version history; specs need archiving too. Code pulls the latest before development; specs need a consistency check before development too.

Let us walk through this process using the follow-up to the OKR case from the specification chapter.

### Starting State

The first feature (OKR basic CRUD) is complete. The main spec contains three sub-features (objective management, KR management, quarterly overview), 5 business rules (BR-001 through BR-005), and 6 API endpoints. The Out of Scope list explicitly excludes 8 capabilities, one of which is "objective alignment."

Now the product team says: the next version needs to support cross-department OKR alignment. Department heads can link their objectives to the objectives of their parent department. This is precisely the capability that was explicitly excluded before.

### Creating a Change Package

Each change is a self-contained feature package, covering every level from user story to acceptance criteria. It is not a code-level diff. It is a complete spec-level description.

```
changes/cross-dept-alignment/
├── story.md          # User story
├── design.md         # Architecture impact analysis
├── delta-spec.md     # Incremental modifications to the main spec
├── tasks.md          # Task breakdown
└── checklist.md      # Acceptance checklist
```

story.md describes the user value: department heads need to link their objectives to those of the parent department when setting quarterly OKRs, so the organization can track alignment from the company level down to the department level.

design.md analyzes the architecture impact: add an alignment module (alignment relation table, CRUD endpoints), modify the existing objective query endpoint (return an alignment status field), modify the objective detail page (display alignment relations).

delta-spec.md is the core. It uses three sections, ADDED, MODIFIED, and REMOVED, to describe incremental modifications to the main spec, with each requirement accompanied by a complete acceptance scenario:

```markdown
## ADDED Requirements

### Feature: Objective Alignment

**User Story**: As a department head, I want to link department objectives
  to parent department objectives so that the organization can track
  objective alignment from company level to department level.

**Acceptance Criteria**:
- [ ] The objective detail page displays an "Aligned to" field for
      selecting parent department objectives
- [ ] After an alignment relationship is established, the quarterly
      overview page displays the alignment chain
- [ ] Only users with the department head role can set alignment
      relationships

**Business Rules**:
| # | Rule | Condition | Outcome |
|---|------|-----------|---------|
| BR-006 | Alignment direction | When setting alignment | Can only align to parent department objectives, not peer or reverse |
| BR-007 | Alignment optional | When creating objectives | Alignment relationship is not required |

**API Contract**:
| Endpoint | Method | URL |
|----------|--------|-----|
| Set alignment | POST | /system/okrAlignment |
| Query alignment chain | GET | /system/okrAlignment/chain |

## MODIFIED Requirements

### Feature: Objective Management

**Change description**: The objective query endpoint adds an alignedTo
field, returning information about the parent objective this objective
is aligned to. The objective list page adds an "Alignment status" column
(aligned / not aligned).

### Out of Scope

**Change description**: Remove the "objective alignment" entry. This
capability is now included in the current development scope.

## UNCHANGED (Explicit Declaration)

BR-002 (maximum 5 KRs per objective), BR-003 (completion rate range
0-100), and other existing business rules are not affected by this change.
```

The UNCHANGED section is the key mechanism for preventing the scenario described in the introduction. Without explicitly declaring "what has not changed," the Agent may inadvertently override existing rules when implementing new features. Explicit declaration turns preserved rules from implicit assumptions into auditable documentation.

### Validation at Creation Time

The change package is validated before any coding begins. This is the lowest-cost interception point: fixing problems only requires changing documents.

Automated validation checks consistency-level issues: whether each MODIFIED requirement in the delta-spec exists in the main spec. Whether new API endpoints have path conflicts with existing ones. Whether the new BR-006 and BR-007 contradict existing BR-001 through BR-005. Whether the affected modules declared in design.md cover all modules involved in the delta-spec changes.

Human review focuses on the intent level: whether the user story accurately reflects product intent. Whether the business rule for alignment direction (only aligning upward) is correct. Whether the MODIFIED scope is complete, with no overlooked affected modules.

After review passes, the change package becomes the input for Agent execution, entering the closed loop established in the specification and verification chapters.

### Execution Loop

The Agent implements features based on the delta-spec and tasks. Verification checks against the checklist and acceptance scenarios. This phase is identical to the previous two chapters and requires no new methods.

### Merge and Post-Merge Verification

After implementation and verification are complete, the delta merges into the main spec. The merge itself is a requirement-granularity operation: ADDED content is appended to the corresponding module in the main spec, MODIFIED content replaces the full content of the original requirement, and REMOVED content is deleted (in this case, removing "objective alignment" from Out of Scope).

A document-level verification runs after the merge, following the same logic as the test infrastructure from the verification chapter. It checks whether the merged main spec contains internal contradictions: whether the new alignment permission model (department heads only) conflicts with the existing objective management permissions (administrators). Whether the return format of the new alignment query endpoint is consistent with existing endpoints. Whether existing BR-001 through BR-005 still exist in their entirety after the merge.

After the merge, project-context is updated: the module list adds the alignment module, the API list adds two endpoints, and the Out of Scope list removes the now-implemented entry.

### Archiving

The entire change package moves to `archive/2026-04-cross-dept-alignment/`. The archived content includes the original user story, architecture analysis, delta-spec, task list, checklist, and execution results. If the alignment feature has issues three months later, this archive provides the complete decision trail: why it was built, how it was designed, and what conditions were verified.

The archiving principle is traceability without polluting the current workspace. The current Agent loads only the main spec and the documents for the feature it is working on, without being overwhelmed by historical archives.

### Sync Check Before the Next Feature

A third feature arrives (say, OKR scoring). Before starting, the Agent reads project-context to understand the current system state, then checks consistency against the main spec.

Even with strict adherence to the change package process, inconsistencies can still emerge: residual drift from Agent execution (the verification chapter discussed how tests and review catch most but not all of it), or missed updates to related modules during delta merging. The sync check is the safety net for these residual issues.

The sync check and creation-time validation form a closed loop: creation-time validation checks consistency between the change package and the main spec, and the sync check verifies consistency between the main spec and the code. These two gates cover both ends of the spec lifecycle. The check results require human judgment: whether a discrepancy is intentional evolution or accidental omission.

The change package process transforms documentation evolution from "remember to update" into a repeatable engineering action. Each change has a complete spec-level description, two verification gates (at creation and after merge), archiving to preserve the decision trail, and sync checks to catch residual drift. The second principle defined in the introduction, continuous evolution, now has a concrete operational method.

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
