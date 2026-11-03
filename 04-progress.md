# 04 — Progress

Last updated: **2026-11-03**.

## Current position

**M0 — planning baseline prepared. Application implementation has not started.**

The frontend, backend, and documentation repositories are accessible. The documentation repository's actual name is `endurance-canva-dosc`. The initial delivery contains these planning documents and an orientation README in each application repository.

This status describes documentation progress only. There is no working studio, model integration, database, deployment, or public learning page yet.

## Milestone status

| Milestone | Status | Evidence / next condition |
| --- | --- | --- |
| M0 — Planning | Complete | [Scope](01-product-and-scope.md), [architecture](02-architecture-and-stack.md), [build checklist](03-features-and-build-plan.md), and [quality/release plan](05-quality-and-publishing.md) |
| M1 — Foundation and interactive lesson | Not started | Scaffold applications; establish schema/catalogue; demonstrate deterministic three-zone scene |
| M2 — Conversational builder | Not started | A prompt and a meaningful refinement produce validated, interactive content |
| M3 — Useful local studio | Not started | Editing, reliable save/reopen, undo, JSON portability, and one PNG export format |
| M4 — Hosted creator studio | Later; not started | Owner access, backend protection, operating limits, and durable storage if required |
| M5 — Public lessons / website | Later; not started | Publish explicitly reviewed revisions with readable pages and shareable URLs |
| M6 — Visitor questions | Later; not started | Bounded questions about supported lessons, with appropriate access and cost controls |

The task checkboxes in [03 — Features and build plan](03-features-and-build-plan.md) are the detailed implementation checklist. Keep this page as the compact milestone summary rather than maintaining a second copy of every task.

## Next concrete task

Start **M1** with a minimal frontend/backend scaffold and one valid example contract. Then render the three-zone scene with fictional LT1 = 180 W and LT2 = 240 W, a power slider, readable labels, and explicit source/assumption space. Verify it on mobile and at the exact boundaries.

Do not spend M1 building a catalogue of lessons. Move into M2 after the first scene works so the conversation-based creation idea is tested early.

## Decisions recorded

| Decision | Baseline |
| --- | --- |
| Repositories | Three separate repos, following Milan's created layout |
| Product / repository naming | Endurance Canvas / existing `endurance-canva-*` URLs |
| Language | Plain JavaScript; English interface/content first |
| First user | Milan as a single creator, initially using a local studio |
| First scene | Explain three zones from supplied fictional thresholds |
| AI boundary | Validated lesson data selecting tested components and approved evidence |
| Initial persistence | Browser drafts plus JSON backup/import |
| Future durable persistence | MongoDB/Mongoose at the hosted studio checkpoint |
| Public use | Reviewed lesson pages first; visitor Q&A later |

## Decisions to make at the relevant milestone

| Decision | When | Blocks current documentation? |
| --- | --- | --- |
| Compatible runtime/package versions | M1 scaffold | No |
| Final palette and visual treatment | M1 first scene | No |
| Claim-level evidence and approved teaching copy | M1 content review | No; required before a lesson is marked reviewed |
| Live model and acceptable usage budget | M2 live-provider evaluation | No; mock mode works first |
| Exact owner-login implementation, hosting plans, database credentials | M4 | No |
| Website path/subdomain and publication workflow | M5 | No |

## Updating progress

For each completed task, tick its checkbox and record a short update here with the task ID, relevant repository commit/PR, what changed, and the verification result. A written plan or mocked provider response does not prove a live feature. If a step is blocked, name the concrete blocker and the next action.

Commit histories: [frontend](https://github.com/mkmilan/endurance-canva-frontend/commits/main), [backend](https://github.com/mkmilan/endurance-canva-backend/commits/main), [documentation](https://github.com/mkmilan/endurance-canva-dosc/commits/main).

## Activity log

### 2026-11-03 — Planning baseline

Prepared the project definition, technology plan, proposed data/API contract, staged feature checklist, progress tracker, and quality/publication plan. Added repository orientation documents. Application work remains unchecked.
