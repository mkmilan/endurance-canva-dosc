# Endurance Canvas — project guide

**Explore, understand, and explain endurance.**

Endurance Canvas helps Milan build interactive explanations of endurance physiology. The first studio combines a guided conversation with an editable three-zone lesson, a working power control, visible assumptions, and sources. Reviewed lessons can later become public learning pages on the coaching website.

**Status: first guided-demo implementation, 3 November 2026.** The frontend and backend are implemented, with browser drafts, editing, revisions, JSON backups, and PNG export. The chat uses prepared content; real AI and MongoDB saving are not enabled. See [Progress](04-progress.md) for the tested release, deployment links, and remaining work.

## Try the studio

Open **[Endurance Canvas](https://endurance-canva-frontend.vercel.app/)**. Choose “Explain three training zones,” explore the suggestion, and click “Use this version.” Then try “Show an example above LT2” or edit the title and fictional thresholds yourself. Save the lesson, reopen the page, and use Export for a JSON backup or PNG visual summary.

The testing URL is accessible without an application login. It exposes the guided demo; the frontend blocks live model generation. Drafts are stored in that browser, so use JSON export to keep a separate copy or move a lesson to another device.

![Verified first studio release](assets/studio-preview.jpg)

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

The implemented stack is plain JavaScript, Next.js, Tailwind, React/SVG visuals, and a separate Node/Express backend. Validated lesson data selects tested components; the diagram responds locally without generation requests. The first version saves drafts in the browser with portable JSON backups. Milan selected Vercel and Render for testing. MongoDB/Mongoose becomes relevant when introducing durable server storage.

## Continuing work

Read [Progress](04-progress.md) first, then the next milestone in [Features and build plan](03-features-and-build-plan.md). Treat unchecked features as planned. Update the checklist and progress when an implementation has evidence behind it; link the relevant frontend/backend commits. Keep each repository independently runnable once its scaffold exists.

The next product checkpoint is **M2's real-model demonstration**: a supported prompt produces a useful lesson, a follow-up meaningfully refines it, and failures preserve the current draft. Before enabling paid generation, finish owner access, select a model and budget, and evaluate its actual outputs. The guided demo is available for testing the studio mechanics first.
