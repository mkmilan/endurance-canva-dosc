# 04 — Progress

Last updated: **2026-11-03**.

## Current position

**The first Endurance Canvas studio is running on Vercel with its separate API on Render.** It supports one interactive three-zone lesson, guided conversation, candidate preview/accept/reject, manual editing, local drafts and revisions, source review, JSON portability, and PNG export.

The conversation is explicitly a **guided demo using prepared teaching content**. No live model is configured or evaluated. The frontend refuses live-generation mode. Atlas access is confirmed, but this release does not connect to MongoDB: drafts live in IndexedDB in the current browser.

## Testing links and deployed services

| Resource | Link / configuration |
| --- | --- |
| Studio | [endurance-canva-frontend.vercel.app](https://endurance-canva-frontend.vercel.app/) |
| Backend health | [endurance-canva-backend.onrender.com/health](https://endurance-canva-backend.onrender.com/health) |
| Vercel project | `endurance-canva-frontend`; project `prj_Ryn4Qhwsf7NXdoWLhTmOAFZ4M3Of`; Node 24; Frankfurt region |
| Frontend deployment | `dpl_F5B45ZH4AXWgwzVG1gudAvUZZbDz`, ready on source revision `a3e182ac8ff9182bf39ca0c3cd2ccbd27e7405c9` |
| Render service | `endurance-canva-backend`; service `srv-dbl0vl6c6qcc73bm1450`; free instance; Frankfurt region |
| Backend deployment | `dep-dbl0vlmc6qcc73bm1500`, live on source revision `5790a2901212591a293313984f81058133cf8d17` |
| Browser verification | [GitHub Actions](https://github.com/mkmilan/endurance-canva-frontend/actions) |

The production testing URL is accessible without an application login. Vercel's existing project protection settings were retained; do not describe them as implemented application owner authentication. The backend requires a service token for catalogue and generation requests, and only the Next.js server holds that token. No paid model endpoint is available through the studio.

Both services are linked to their respective GitHub repositories and configured to deploy `main`. Vercel's automatic deployment was observed after the code and CI commits. Render's initial deployment was verified at the published backend revision. Secrets are configured in the hosting services and are absent from source control.

## Milestone status

| Milestone | Status | Evidence / next condition |
| --- | --- | --- |
| M0 — Planning | Complete | Scope, architecture, build checklist, progress, and quality documents are maintained in this repository |
| M1 — Foundation and interactive lesson | Complete | Canonical schema/catalogue, deterministic zone display, responsive studio, local saves, and reviewed source limitations |
| M2 — Conversational builder | Partial: guided demo works | Real model selection, credentials, owner access, usage accounting, and actual output evaluation remain |
| M3 — Useful local studio | Implemented and verified; Milan tryout pending | All technical browser checks passed; Milan's first real-use tryout remains |
| M4 — Hosted creator studio | Testing deployments live; owner/cloud features pending | Backend service protection exists; application owner access, persistent budgets, and optional MongoDB storage remain |
| M5 — Public lessons / website | Not started | Publish explicitly reviewed revisions with readable pages and shareable URLs |
| M6 — Visitor questions | Not started | Bounded questions about supported lessons, with appropriate access and cost controls |

Detailed feature checkboxes live in [03 — Features and build plan](03-features-and-build-plan.md).

## Verification record

- **Backend:** 18 passing `node:test` cases, syntax checks, and whitespace checks. Coverage includes schema/semantic validation, exact LT1/LT2 boundaries, authentication, guided creation/refinement, unsupported requests, request/rate limits, invalid provider output, errors, and timeouts.
- **Frontend:** 15 passing `node:test` cases and a successful production build. Coverage includes revision preservation, review invalidation, undo, JSON round trip, unsupported versions, imported evidence reconciliation, invalid content, and local/hosted origin validation.
- **Published source:** all application files were compared byte-for-byte with the remote GitHub commits. Frontend contracts are pinned to the clean backend revision and include SHA-256 hashes.
- **Render:** `/health` returned HTTP 200 with `mode: demo`. Unauthenticated catalogue and generation calls returned HTTP 401. After the proxy fix, a same-origin request through Vercel returned HTTP 200 with a valid demo lesson; a foreign-origin request returned HTTP 403.
- **Live browser:** the Vercel studio connected to Render. A three-zone request returned a labelled suggestion; accepting it created a revision. “Show an example above LT2” changed the starting power to 295 W and Zone 3. Saving and reloading restored the content, conversation, and revision.
- **Image export:** the hosted app downloaded a readable 1080 × 1350 PNG with title, zone graphic, fictional-input explanation, and qualified attribution. The actual image was inspected. See [the exported example](assets/lesson-export.png) and [studio screenshot](assets/studio-preview.jpg).
- **Automated browser flow:** all 11 browser checks passed in [GitHub Actions run 41682209486](https://github.com/mkmilan/endurance-canva-frontend/actions/runs/41682209486), using frontend revision `a3e182a` and backend revision `5790a29`. The checks exercised exact boundaries and keyboard input, guided creation/refinement, unsupported requests, rapid edit/switch persistence, save/reload, review/import semantics, invalid import preservation, duplicate/undo, PNG download, and 360 px canvas/conversation layouts without horizontal overflow. There were no uncaught page errors or console errors. The mobile screenshots were also inspected visually. The [full browser report](verification/2026-11-03-browser-report.json) and [mobile preview](assets/studio-mobile.png) are preserved here; the workflow retains detailed logs/traces for 14 days.

These checks do not establish live-AI quality, cross-device persistence, independent scientific approval, or completion of Milan's own usability review.

## Next concrete task

Try one explanation in the deployed studio and use the feedback to shape **M2's real-model path**. Keep the first lesson type bounded while adding:

1. Verified owner access for generation, with both frontend and backend access checked.
2. A selected model, server-side credentials, and an agreed usage budget with persistent accounting.
3. A small evaluation set for useful creation/refinement, unsupported questions, and source-grounded explanations.

When cloud drafts become useful, add a dedicated Endurance Canvas Atlas database and ownership/revision rules. Keep its collections and credentials separate from other projects.

## Decisions recorded

| Decision | Current implementation |
| --- | --- |
| Repositories | Separate frontend, backend, and documentation; preserve the actual `-dosc` spelling |
| Language | Plain JavaScript; English interface/content |
| First scene | Fictional LT1 180 W, LT2 240 W, 0–350 W display, authored starting power 200 W |
| Display boundaries | Z1 below LT1; Z2 includes both exact boundaries; Z3 above LT2; equality gets an explicit label |
| Generation | Clearly labelled prepared demo; optional backend live adapter remains unconfigured/unevaluated and blocked by the frontend |
| Initial persistence | IndexedDB browser drafts, 12 previous revisions, JSON backup/import |
| Exports | Versioned JSON with trusted evidence snapshot; one 1080 × 1350 PNG summary |
| Review | Local creator review of a revision; edits, undo, and imports require fresh review |
| Hosting | Vercel frontend, Render free backend, both in Frankfurt |
| Atlas | Access confirmed; no app database provisioned or integrated |
| Public use | Reviewed learning pages first; website connection and visitor Q&A later |

## Source revisions

- [Backend implementation — 5790a29](https://github.com/mkmilan/endurance-canva-backend/commit/5790a2901212591a293313984f81058133cf8d17)
- [Frontend implementation — 0ef3813](https://github.com/mkmilan/endurance-canva-frontend/commit/0ef38134aae49ea0ea676b6b20003f6e49d0b760)
- [Verified proxy — 355a25a](https://github.com/mkmilan/endurance-canva-frontend/commit/355a25a32cac07731a027e5bae1ebdd563c843c4)
- [Passing browser workflow — a3e182a](https://github.com/mkmilan/endurance-canva-frontend/commit/a3e182ac8ff9182bf39ca0c3cd2ccbd27e7405c9)

Commit histories: [frontend](https://github.com/mkmilan/endurance-canva-frontend/commits/main), [backend](https://github.com/mkmilan/endurance-canva-backend/commits/main), [documentation](https://github.com/mkmilan/endurance-canva-dosc/commits/main).

## Activity log

### 2026-11-03 — First testing implementation

Built and published the backend and frontend, configured the testing deployments, verified the live guided-demo path, and added repeatable browser checks. Corrected contract-boundary parity, removed stale wattages from assumptions, protected pending saves when switching lessons, guarded stale generation callbacks, and improved phone chart labels and PNG layout. Updated the project plan to distinguish tested functionality from later real-AI, owner-access, and cloud-storage work.

### 2026-11-03 — Planning baseline

Prepared the project definition, technology plan, shared contract strategy, staged checklist, progress tracker, and quality/publication plan. Established repository orientation documents.
