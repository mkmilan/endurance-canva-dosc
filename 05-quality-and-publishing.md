# 05 — Quality, verification, and later publishing

First implementation baseline: **2026-11-03**. This document separates checks already performed on the initial backend from acceptance requirements for the studio and later releases. The current deployment and browser-verification record is maintained in [04 — Progress](04-progress.md).

## 1. Scientific quality

Each interactive lesson must separate supplied example inputs, deterministic display rules, evidence-supported explanations, and modelling assumptions.

For the first scene, 180 W and 240 W are fictional thresholds. The app does not calculate a person's LT1/LT2, infer a lactate curve, equate a fixed lactate concentration with a universal threshold, or predict training adaptations. Changing those inputs demonstrates the chosen boundaries.

A small versioned evidence catalogue is implemented in the backend's `contracts/catalog.v1.json` and synced to the frontend. Source entries record identity, bibliographic metadata, the particular claim supported, qualifications, and review state; claim cards have their own IDs. A valid citation ID alone does not establish that a statement is supported. Guided demo content uses these fixed cards. The optional live adapter supplies the same catalogue to the model, but its outputs still require review.

The first reference is [Seiler and Kjerland (2006), Quantifying training intensity distribution in elite endurance athletes](https://pubmed.ncbi.nlm.nih.gov/16430681/). The study delineated three zones with **ventilatory thresholds**. That limited claim and its qualification were checked for the first catalogue; the source does not validate the fictional wattages, a lactate-threshold algorithm, or equivalence between ventilatory and lactate thresholds. The demonstration explains supplied boundaries using an explicit display rule. Broader physiological LT1/LT2 teaching claims need appropriate evidence before they are added or approved for publication.

AI and manual edits create draft revisions. A creator can review a specific revision after checking its claims, assumptions, labels, and source links. Any later content change requires a new review. An illustrative source-free draft is acceptable during development, but cannot be treated as ready for public publication.

Imported JSON also starts unreviewed. Local review metadata helps the creator keep track; when publication is implemented, the server must record approval of the exact stored revision rather than trusting a review flag supplied in a file or model response.

## 2. Verification that matters

The initial backend passed **18 `node:test` tests**, plus syntax and whitespace checks. Tests cover the canonical fixture, schema/semantic parity, exact boundaries, malformed content, protected routes, guided creation/refinement, unsupported questions, request/rate limits, provider errors, and timeouts. Ajv validates the JSON-schema side of the contract tests; a shared dependency-free semantic validator runs in the applications. These tests do not demonstrate live-model quality or a working remote deployment.

The table below remains the acceptance checklist for the relevant implementation stage. Mark an item complete only when its own evidence is recorded; future cloud/publication checks are not claimed as passed.

| Area | Check | Passing behaviour |
| --- | --- | --- |
| Zone boundaries | Values below, equal to, between, and above the two thresholds | Matches the documented convention; exact boundaries are labelled |
| Input validation | Reversed/equal thresholds, out-of-range values, missing units, non-finite numbers | Clear feedback; last valid view remains usable |
| Rendering | Unsupported component, executable fields, unsafe markup, unsupported schema version | Rejected; no generated code or arbitrary HTML executes |
| Evidence | Unknown source/claim IDs, unsupported scientific assertions | Rejected or kept explicitly unreviewed; no fabricated citation |
| Conversation | Representative creation prompts and meaningful refinements | Supported request changes the intended content; unsupported requests are explained |
| Failure recovery | Timeout, invalid model output, provider failure, stale response | Existing draft survives; user sees what happened and can retry |
| Persistence | Save, reload, revision undo, JSON round trip | Preserves content, defaults, versions, and evidence references |
| Evidence history | Reopen/export/import an older lesson after a catalogue update | Resolves its recorded catalogue version; imported metadata never grants approval |
| Viewer state | Move a slider without applying a new default | The saved authored lesson remains unchanged |
| Mobile and keyboard | 360 px viewport, touch controls, keyboard focus, non-colour labels | No horizontal overflow; controls and feedback remain usable |
| Image export | One fixed export format, long title, source footer | Downloaded PNG is readable, correctly sized, and not clipped |
| Hosted access | Unauthenticated requests to frontend and direct backend URLs | Creator operations and paid generation are protected |
| Future cloud saves | Concurrent edits with an old expected revision | Explicit conflict instead of a silent overwrite |
| Future publication | Edit a published lesson's draft | Published approved revision does not change until explicitly replaced |

Extend the existing `node:test` suites for meaningful persistence/revision and contract changes. Browser checks must exercise the core creation/edit/save/export flow and mobile layout. Inspect the first exported image visually. Use a small labelled evaluation set when live AI is introduced; structural validity is not a scientific quality score. Record exactly which local, browser, and deployed checks ran in Progress.

## 3. AI use and operating limits

The testing configuration uses **`AI_MODE=demo`**: prepared teaching content and supported editing rules, clearly labelled “Guided demo.” It makes no model calls. The backend includes an optional live adapter using AI SDK 7.0.162 and AI Gateway, but no model/key has been configured or real-model evaluation completed. The frontend also blocks forwarding generation when the backend reports live mode. Select the live model and check current pricing only when owner access and evaluation are ready.

The implemented backend bounds JSON request size (32 KiB), prompt length (4000 characters), generation time, simultaneous requests, and total attempts per minute. Defaults are 25 seconds, two simultaneous generations, and 12 attempts per minute. The optional live adapter limits output and sets SDK retries to zero. These counters are global to one process and reset on restart; they are not persistent daily quotas or exact spending caps. Add persistent accounting and an explicit budget before broader live-AI use.

Sliders, reopening drafts, saving, and exporting should work without an AI request. A configured model/provider failure must preserve local work. Logs should capture request IDs, timings, validation/error codes, and usage without storing secret keys or full prompt contents by default.

## 4. Testing hosting and the later owner checkpoint

Milan selected Vercel for frontend testing and Render for the backend. The frontend project is connected with its default deployment-protection/SSO configuration retained. The Render free backend is live, with successful health and unauthenticated-access checks recorded in Architecture and Progress. This testing release is configured for guided demo use through the studio; M4's owner-login and durable-storage requirements remain future work. Record the frontend deployment outcome and effective access behaviour separately in Progress.

The backend's `/api` routes now require a server-only service token in production. Its public `/health` response contains only health/version/mode information. The frontend proxy keeps the token out of browser code. Before enabling live generation, implement and verify creator access on the frontend as well; backend service authentication alone is not end-user authentication.

Cloud saving remains future work. Atlas access has been confirmed, but no Endurance Canvas database has been provisioned or integrated. Introduce MongoDB/Mongoose with server-assigned ownership, revision conflicts, export/recovery behaviour, and dependable deletion when durable drafts are needed. Keep all provider and database credentials server-side.

## 5. Public learning pages — M5

Publish an explicitly approved, fixed lesson revision. The public page should include a readable introduction, interactive component, assumptions, sources, author/date, and a relevant link to Milan's coaching. Retain a stable URL for each topic. Select the website path and deployment integration at M5; the standalone studio can remain the authoring tool.

Prefer rendering the public explanation as ordinary server-rendered HTML alongside the interactive component. Use crawlable links, descriptive page metadata, a canonical URL, and a sitemap for the published collection. [Google's JavaScript SEO documentation](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics) explains crawling, rendering, and indexing. Discoverable pages create an opportunity for search traffic; they do not guarantee rankings or coaching enquiries.

Keep unpublished drafts and creator routes out of the public collection. Search-engine exclusion is not access protection. A limited review of public data handling and service terms belongs at this publishing milestone, based on what the product actually does.

## 6. Visitor questions — M6

Initially answer questions about the supported lesson catalogue using the same reviewed components and evidence. Distinguish an exploratory AI response from the fixed reviewed lesson. Never let a visitor modify the published original. Add per-visitor/global cost controls and clear recovery for unsupported questions before opening this feature publicly.

Broader physiology topics, fresh literature retrieval, personalised athlete interpretation, and arbitrary generated widgets each require their own scope and evaluation. They are future extensions, not implied capabilities of the first release.
