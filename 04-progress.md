# 04 — Progress

Last updated: **2026-11-04**.

## Current position

**The creator-access release is deployed; the public studio remains a working guided demo.** The release adds private owner setup, email/password sign-in, password change, sign-out, explicit Guided demo / Live AI selection, persistent attempt limits, and a validated live-model adapter. Hosted account activation and real-model evaluation are still pending.

The existing three-zone lesson, candidate preview/accept/reject, manual editing, revision history, source review, JSON portability, and PNG export remain available. Lessons stay in IndexedDB on the current browser. Signing in does not upload them.

## Hosted services and latest source

| Resource | Verified state |
| --- | --- |
| Studio | [endurance-canva-frontend.vercel.app](https://endurance-canva-frontend.vercel.app/) |
| Creator access | [/creator](https://endurance-canva-frontend.vercel.app/creator); currently explains that the connection is pending |
| Backend | [Health endpoint](https://endurance-canva-backend.onrender.com/health); HTTP 200, version `0.2.0`, mode `demo` |
| Frontend source | [68ecb55](https://github.com/mkmilan/endurance-canva-frontend/commit/68ecb558198f287b6b51afb5a01a00c770b88d25); pins the corrected backend, fixes the mobile test wait, and explicitly marks creator access noindex |
| Vercel deployment | `dpl_GmhkKCQArr9Xe1bcicQkdMvVZoJo`, READY on frontend `68ecb55` |
| Backend source | [259760b](https://github.com/mkmilan/endurance-canva-backend/commit/259760be5e75e4690765d679085933de181561ba); fixes the setup race and uses supported MongoDB timeouts |
| Render deployment | `dep-dbl24ch5efls73f8br90`, LIVE on backend `259760b`; existing free Frankfurt service `srv-dbl0vl6c6qcc73bm1450` |
| Atlas | New project `6aea1c1f1c91a0b9011d4027`; cluster `endurance-canvas`, Free, AWS Frankfurt, MongoDB `8.0.36` |

The Next.js server retains the service credential. Creator routes are restricted to the canonical studio origin; previews remain demo-only. The backend checks owner sessions and reserves usage before paid generation. A configured model ID alone does not enable public model calls.

## Milestone status

| Milestone | Status | Remaining condition |
| --- | --- | --- |
| M0 — Planning | Complete | Keep the plan and verification record current |
| M1 — Foundation and lesson | Complete | Canonical contract and deterministic display retained |
| M2 — Conversational builder | Partial | Guided demo works; live adapter is implemented, but real-model access and seven-case evaluation remain open |
| M3 — Useful local studio | Implemented and verified in current CI | Creator navigation save fix covered; Milan's real-use tryout remains |
| M4 — Hosted creator studio | Owner implementation deployed and verified in CI; hosted activation pending | Connect Atlas, create the owner, and verify hosted sessions/limits; cloud lessons remain later |
| M5 — Public lessons / website | Not started | Explicitly reviewed publishing and shareable pages |
| M6 — Visitor questions | Not started | Bounded questions with access and cost controls |

See [03 — Features and build plan](03-features-and-build-plan.md) for individual checkboxes.

## Current verification record

- **Frontend:** 28 unit/proxy tests pass, production build passes, and whitespace checks pass. Coverage includes canonical-origin isolation, trusted IP forwarding, all session cookies, token redaction, body bounds, backend errors, and demo/live separation.
- **Hosted browser:** verified guided generation, the creator connection-pending page, and return to the studio with the local draft preserved. Screenshots: [current studio](assets/creator-release-studio.jpg) and [creator pending](assets/creator-pending.jpg).
- **Hosted HTTP:** health returns 200; `/api/creator` reports `auth.configured: false`; foreign-origin requests return 403; direct backend creator access without the service credential returns 401.
- **Current CI passed:** [run 41690421183](https://github.com/mkmilan/endurance-canva-frontend/actions/runs/41690421183) succeeded at every step on frontend `68ecb55` and backend `259760b`. The run passed **55 backend tests, including seven real-MongoDB integration cases**, and **28 frontend tests**, then completed the production browser workflow. The earlier kernel guard and concurrent-setup response mismatch are resolved; supported MongoDB timeouts are now configured.
- **Browser suite:** **18/18 flow checks passed** against the production frontend build and an isolated MongoDB replica set. The [creator browser report](verification/2026-11-03-creator-browser-report.json) confirms the exact source revisions and no page or console errors. Model responses were explicitly labelled deterministic fixtures; **no real AI provider was called**.
- **CI visual review:** the [signed-in account](assets/creator-signed-in-ci.png), [360 px account page](assets/creator-mobile-ci.png), and [studio with an AI test-fixture suggestion](assets/studio-live-fixture-ci.png) were inspected and showed no clipping. These screenshots document the local CI fixture, not activated hosted accounts or real-model output.

The release also fixes immediate Creator navigation losing an edit before the autosave debounce: navigation now waits for a successful save. A backend regression covers cancelling a request during session lookup so generation does not start afterward.

### Prior release — historical evidence

The first demo release passed 18 backend tests, 15 frontend tests, and **11 browser checks** in [run 41682209486](https://github.com/mkmilan/endurance-canva-frontend/actions/runs/41682209486), using frontend `a3e182a` and backend `5790a29`. Its [browser report](verification/2026-11-03-browser-report.json), [mobile view](assets/studio-mobile.png), and [exported PNG](assets/lesson-export.png) remain historical evidence, not verification of the new owner flow.

## Activation still required

Render has `AUTH_SECRET`, `STUDIO_ORIGIN`, `MONGODB_DATABASE`, the candidate model `google/gemini-3.5-flash-lite`, and limits of **20 attempts per UTC day / 200 per UTC month** configured. `AI_MODE` remains `demo`. Vercel has `STUDIO_ORIGIN` configured.

On **2026-11-04**, Milan supplied Render outbound ranges `74.220.51.0/24` and `74.220.59.0/24` and explicitly approved the database work. The connector still refused both access-list and database-user creation with “User did not confirm…”. Read-only checks confirmed zero users and zero access-list entries. No database URI was generated or Render setting changed. Further permission in chat is not missing; the connector confirmation flow remains blocked. Gateway key management is unavailable through the connected MCP tools, and the CLI/dashboard browser are logged out. **No real model calls or billing changes have occurred.**

Follow [06 — Creator and AI activation](06-creator-and-ai-activation.md): connect the isolated database, issue the private owner setup link, verify hosted access, then connect the Gateway key and evaluate the model. The backend's [seven real-model evaluation cases](https://github.com/mkmilan/endurance-canva-backend/blob/main/test/live-evaluation-cases.json) explicitly remain **`not-run`**. Fixture success cannot establish model quality, actual provider cost, or Render-to-Atlas connectivity.

Cloud lesson storage, website embeds, public publishing, and visitor questions remain separate later work. Source histories: [frontend](https://github.com/mkmilan/endurance-canva-frontend/commits/main), [backend](https://github.com/mkmilan/endurance-canva-backend/commits/main), [documentation](https://github.com/mkmilan/endurance-canva-dosc/commits/main).
