# Endurance Canvas — project guide

**Explore, understand, and explain endurance.**

Endurance Canvas helps Milan build interactive explanations of endurance physiology. The studio combines a guided conversation with an editable three-zone lesson, a working power control, visible assumptions and sources.

**Current home: [Endurance Canvas on ChatGPT Sites](https://endurance-canvas.quirkychick26.chatgpt.site).** Version 1 is published privately for Milan as of 4 November 2026. The frontend, API and D1 usage database run together. Native Sites sign-in replaces the unfinished MongoDB owner-account setup. Guided creation and all local studio features work; real AI still needs its existing Gateway key transferred and evaluated.

## Drafts and the old app

Lessons remain saved in the browser. The old and new addresses have separate browser storage. On the [old Vercel studio](https://endurance-canva-frontend.vercel.app/), choose **My lessons → Back up library**; on Sites, choose **My lessons → Restore library**. The complete backup includes history and conversations. Existing lessons are preserved on restore; imported lessons need a fresh review.

The old Vercel/Render/Atlas resources remain intact until wanted drafts are transferred and the remaining activation gate is resolved. Only this app's dedicated resources are in the retirement scope. Other projects and all GitHub source history are preserved.

## Project documents

| Document | Purpose |
| --- | --- |
| [01 — Product and scope](01-product-and-scope.md) | Product goals and first useful lesson |
| [02 — Architecture and stack](02-architecture-and-stack.md) | Current Sites architecture and historical design |
| [03 — Features and build plan](03-features-and-build-plan.md) | Product milestones and checkboxes |
| [04 — Progress](04-progress.md) | Current verified state and remaining work |
| [05 — Quality and publishing](05-quality-and-publishing.md) | Scientific review and product quality |
| [06 — Creator and AI activation](06-creator-and-ai-activation.md) | Current identity, secret and model activation steps |
| [07 — Sites migration](07-sites-migration.md) | Transfer plan, implementation, evidence and scoped retirement |

The new runtime source is stored with the Site. The historical repositories remain [frontend](https://github.com/mkmilan/endurance-canva-frontend), [backend](https://github.com/mkmilan/endurance-canva-backend) and this [documentation repository](https://github.com/mkmilan/endurance-canva-dosc). Keep the existing `endurance-canva-*` spelling in their URLs.

## Current verification

54 Sites tests pass, the production Worker build passes, and production D1/API checks pass. Browser QA covered guided create/refine, saving, whole-library restore without overwrite, restored undo history, editing, account navigation, reload and PNG export. The old frontend backup update passes its 42-test suite and production build and is deployed. See [Progress](04-progress.md) for exact release identifiers and verification limits.

The next product gate remains actual model evaluation. Cloud lesson synchronization, public publishing, website embeds and visitor questions remain future work.
