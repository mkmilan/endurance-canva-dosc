# Endurance Canvas — project guide

**Explore, understand, and explain endurance.**

Endurance Canvas will help Milan create interactive explanations of endurance physiology through conversation. A prompt becomes an editable lesson with a working visual, appropriate controls, assumptions, and evidence. Reviewed lessons can later become public learning pages on the coaching website.

**Status: planning baseline, 3 November 2026. Application implementation has not started.** These repositories contain project documentation; there is no running application, configured AI service, or deployment yet.

## Read these five documents

| Document | What it answers |
| --- | --- |
| [01 — Product and scope](01-product-and-scope.md) | Who the app serves, how it should feel, and what belongs in the first version. |
| [02 — Architecture and stack](02-architecture-and-stack.md) | Technologies, repository responsibilities, AI/data flow, and the proposed API contract. |
| [03 — Features and build plan](03-features-and-build-plan.md) | Ordered milestones, feature checkboxes, and acceptance criteria. |
| [04 — Progress](04-progress.md) | What is actually complete, what is next, and decisions still to make. |
| [05 — Quality and publishing](05-quality-and-publishing.md) | Scientific review, practical verification, operating limits, and the later website/public release. |

## The three repositories

| Repository | Responsibility |
| --- | --- |
| [endurance-canva-frontend](https://github.com/mkmilan/endurance-canva-frontend) | Mobile-friendly studio, conversation interface, interactive renderer, local drafts, and exports. |
| [endurance-canva-backend](https://github.com/mkmilan/endurance-canva-backend) | Validated generation/refinement, trusted lesson/source catalogues, and later durable storage. |
| [endurance-canva-dosc](https://github.com/mkmilan/endurance-canva-dosc) | Canonical product plan, architecture decisions, feature checklist, and progress. |

The product name is **Endurance Canvas**. Repository URLs retain the existing `endurance-canva-*` spelling, including `-dosc`. Use these exact URLs in links and Git remotes.

## First useful result

Milan asks for a beginner explanation of three training zones, explores an interactive example, requests a meaningful refinement, and saves a version worth sharing. Start with one excellent lesson type. More lesson types extend the same studio after that full workflow works.

The suggested stack is plain JavaScript, Next.js, Tailwind, React visuals, and a separate Node/Express backend. AI proposes structured lesson content; tested components determine the visual behaviour. The first version saves drafts in the browser with portable JSON backups. MongoDB/Mongoose becomes relevant when introducing durable server storage.

## Continuing work

Read [Progress](04-progress.md) first, then the next milestone in [Features and build plan](03-features-and-build-plan.md). Treat unchecked features as planned. Update the checklist and progress when an implementation has evidence behind it; link the relevant frontend/backend commits. Keep each repository independently runnable once its scaffold exists.

The next development task is **M1: establish the lesson contract and build the first deterministic interactive scene**. The first complete product demonstration is **M2: prompt, interactive answer, and useful conversational refinement**.
