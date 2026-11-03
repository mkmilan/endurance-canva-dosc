# 05 — Quality, verification, and later publishing

Planning baseline: **2026-11-03**. These are focused product requirements for the relevant milestones. They are not completed application checks or a legal-policy package.

## 1. Scientific quality

Each interactive lesson must separate supplied example inputs, deterministic display rules, evidence-supported explanations, and modelling assumptions.

For the first scene, 180 W and 240 W are fictional thresholds. The app does not calculate a person's LT1/LT2, infer a lactate curve, equate a fixed lactate concentration with a universal threshold, or predict training adaptations. Changing those inputs demonstrates the chosen boundaries.

Maintain a small versioned evidence catalogue. Each entry records a source ID, title, authors/year, DOI or canonical URL, the particular claim it supports, qualifications, and review state. Each claim has its own ID and references. A valid citation ID alone does not establish that a statement is supported: review the relationship between the claim and the source. The backend supplies relevant approved claim cards to the model, and resolves their references to trusted metadata.

One starting reference is [Seiler and Kjerland (2006), Quantifying training intensity distribution in elite endurance athletes](https://pubmed.ncbi.nlm.nih.gov/16430681/). The study delineated three zones with **ventilatory thresholds**. It can inform the three-zone teaching framework; it does not validate the fictional wattages, a particular lactate-threshold algorithm, or a claim that ventilatory and lactate thresholds are interchangeable. Review appropriate lactate-specific material before approving LT1/LT2 explanatory claims. Scientific content review is an open M1 task.

AI and manual edits create draft revisions. A creator can review a specific revision after checking its claims, assumptions, labels, and source links. Any later content change requires a new review. An illustrative source-free draft is acceptable during development, but cannot be treated as ready for public publication.

Imported JSON also starts unreviewed. Local review metadata helps the creator keep track; when publication is implemented, the server must record approval of the exact stored revision rather than trusting a review flag supplied in a file or model response.

## 2. Verification that matters

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

Add automated tests for calculation boundaries, contracts, persistence/revision failures, and the core browser flow as those features are built. Use a small labelled evaluation set for AI generation; structural validity is not a scientific quality score. Inspect the first exported image visually. Documentation-only commits require link/format checks, not a fictional application test report.

## 3. AI use and operating limits

Real model calls happen only on the backend. Start with a clearly labelled mock provider; select the live model after comparing its output on the initial lesson and refinements. Recheck current provider pricing at that point. A personal chat subscription is not the app's model configuration.

Bound prompt/context size, output size, elapsed time, concurrent generations, and requests per creator. Configure the SDK's retry behaviour so failures cannot produce an unbounded retry loop. Retrying invalid output should be limited and visible in usage accounting. Use server-side counters; persistent counters are required for a hosted daily quota that must survive restarts. Counts control usage, but are not a guarantee of an exact monetary spend cap.

Sliders, reopening drafts, saving, and exporting should work without an AI request. A configured model/provider failure must preserve local work. Logs should capture request IDs, timings, validation/error codes, and usage without storing secret keys or full prompt contents by default.

## 4. Hosted creator checkpoint — M4

Before exposing the studio remotely, protect creator sessions and the separately reachable backend. Configure usage limits, HTTPS, restricted service credentials, and dependable storage. Keep gateway/database credentials server-side. Decide backup/export retention for durable drafts and verify that deletion removes the relevant stored content.

Use a dedicated project configuration. Confirm current hosting plans, costs, supported runtimes, and relevant service terms when choosing the deployment. No hosting subscription, model credit, database, domain, or deployment is part of this planning commit.

## 5. Public learning pages — M5

Publish an explicitly approved, fixed lesson revision. The public page should include a readable introduction, interactive component, assumptions, sources, author/date, and a relevant link to Milan's coaching. Retain a stable URL for each topic. Select the website path and deployment integration at M5; the standalone studio can remain the authoring tool.

Prefer rendering the public explanation as ordinary server-rendered HTML alongside the interactive component. Use crawlable links, descriptive page metadata, a canonical URL, and a sitemap for the published collection. [Google's JavaScript SEO documentation](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics) explains crawling, rendering, and indexing. Discoverable pages create an opportunity for search traffic; they do not guarantee rankings or coaching enquiries.

Keep unpublished drafts and creator routes out of the public collection. Search-engine exclusion is not access protection. A limited review of public data handling and service terms belongs at this publishing milestone, based on what the product actually does.

## 6. Visitor questions — M6

Initially answer questions about the supported lesson catalogue using the same reviewed components and evidence. Distinguish an exploratory AI response from the fixed reviewed lesson. Never let a visitor modify the published original. Add per-visitor/global cost controls and clear recovery for unsupported questions before opening this feature publicly.

Broader physiology topics, fresh literature retrieval, personalised athlete interpretation, and arbitrary generated widgets each require their own scope and evaluation. They are future extensions, not implied capabilities of the first release.
