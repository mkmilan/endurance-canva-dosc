# Features and build plan

Updated: 2026-11-03. The guided-demo studio and owner-access implementation are deployed for testing. Check a task only after its implementation and relevant verification are complete; record evidence and milestone status in [04-progress.md](04-progress.md). Demo responses do not complete the real-model gate in M2.

The feature checklists live here. The progress document owns the milestone summary and current next action, avoiding competing status lists.

## M0 — Planning and repository setup

Establish the three existing repositories, product scope, proposed stack, contract ownership, build order and verification approach. The documentation repository is `mkmilan/endurance-canva-dosc`; retain that exact repository name in links and commands.

**Gate:** the documents agree on the first lesson, scope and repository responsibilities. Planning completion does not imply that an application exists.

## M1 — Foundation and deterministic lesson

- [x] **M1.1** Scaffold the frontend and backend using plain JavaScript, with documented local commands, configuration examples and dependency lockfiles.
- [x] **M1.2** Establish a versioned lesson contract, component registry and source registry with a single canonical owner. Define required fields, allowed settings and supported source claims.
- [x] **M1.3** Build the three-zone scene using fictional LT1 180 W, LT2 240 W, a 0–350 W display and starting power 200 W.
- [x] **M1.4** Add a mobile studio shell with conversation and preview areas, readable labels, keyboard-operable controls and visible action feedback.
- [x] **M1.5** Save and restore the current draft in the browser, including an honest warning if browser storage fails. Make reset explicit.
- [x] **M1.6** Review the first lesson's teaching copy and evidence. Record the claims each source supports and the limitations of the example before making that copy available to generation.

**Gate:** the deterministic lesson works at 360 px width without horizontal scrolling. Changing power immediately updates its marker and zone description without AI. Threshold order and finite numeric values are validated. Exact LT1/LT2 values display boundary labels. An ordinary reload restores a saved draft. Colour is supplemented by text.

This milestone proves the renderer and contract. Keep it short enough that the conversational product is tested next.

## M2 — Conversational creation and refinement

- [x] **M2.1** Use a clearly identified mock AI response to connect request, validation, preview and failure handling locally.
- [ ] **M2.2** Connect a real, configured AI model through the backend. Keep credentials server-side and bound request size, response size and generation time.
- [ ] **M2.3** Create a lesson from a supported prompt, then refine the current lesson using the conversation and current specification as context.
- [x] **M2.4** Reject invalid specifications, unknown source IDs and unsupported component types. Preserve the current lesson on failure and explain what happened.
- [x] **M2.5** Handle unsupported topics honestly, show progress, prevent accidental duplicate submissions and let the user retry a failed request.

**Current state:** the guided-demo create/refine path works. The live adapter now accepts natural creation/refinement requests, uses a validated supported/unsupported response, and has SDK-level fixture tests. Seven real-model evaluation cases are prepared. The real-provider tasks M2.2 and M2.3 stay open until a credential is connected and actual model outputs are evaluated.

**Gate — first product proof:** “Explain three training zones to a beginner cyclist” produces a valid lesson through a real model call. “Make it simpler and explain what happens at LT1” makes a meaningful revision. The selected model passes representative creation and refinement examples. Timeout and invalid-output checks leave the previous draft usable. Scientific claims still require review; schema validation is not scientific verification.

## M3 — Useful local studio

- [x] **M3.1** Edit the title, explanation, supported visual settings and fictional thresholds directly. Content changes return a reviewed draft to an unreviewed state.
- [x] **M3.2** List, name, save and reopen local lessons. Make a duplicate without changing the original.
- [x] **M3.3** Restore the previous successful revision, including its text and inputs. Keep ordinary slider exploration separate from committed author changes.
- [x] **M3.4** Export and import versioned JSON with its evidence snapshot. Validate imports, preserve the existing lesson on failure and explain unsupported schema versions or unresolved catalogue references.
- [x] **M3.5** Export one polished PNG format with a readable title, visual, example label and attribution. The JSON lesson retains its full source notes.
- [x] **M3.6** Let the creator mark a specific revision reviewed after checking its sources and assumptions. Imported lessons start unreviewed; editing reviewed content requires a fresh review.

**Gate:** a lesson survives a save/reopen and JSON round trip with the same content, inputs and source IDs. Undo restores a useful earlier revision. PNG export matches the intended preview without clipped text. Milan completes one explanation he would use. Local browser storage is clearly distinguished from a backup; export is available for keeping a separate copy.

**User checkpoint:** technical acceptance does not replace Milan trying one explanation he wants to keep.

## M4 — Optional hosted owner studio

- [x] **M4.1** Verify one-owner setup, sign-in/out, password change and server-enforced live-generation access against a real database in CI; keep the demo and local editor open. Hosted activation is recorded separately in M4.4.
- [x] **M4.2** Verify persistent daily/monthly request allowances, idempotency, configured origins, safe errors and minimal usage metadata. A separate Gateway key and spend budget remain activation tasks.
- [ ] **M4.3** Add MongoDB for durable drafts and revisions if cross-device saving is needed; document retention, export and recovery behaviour.
- [ ] **M4.4** Verify save/reopen and AI generation in the hosted environment, including rejection of unauthorised requests, and configure a dedicated Gateway key with an appropriate spend budget.

**Current state:** owner access and persistent attempt allowances passed CI verification, including real MongoDB transactions and browser flows. A dedicated Atlas Free cluster exists, but hosted activation still requires provisioning the database user and network access, plus the AI Gateway credential. Milan authorized the database work and supplied the Render CIDRs on 2026-11-04; the Atlas connector still refuses those writes. Cloud drafts are a separate future task. See [Activation](06-creator-and-ai-activation.md).

**Gate:** the owner completes the creation flow; unauthorised callers cannot read drafts, modify lessons or use the paid AI endpoint. Hosting and model choices are recorded before deployment.

## M5 — Approved public lessons and website connection

- [ ] **M5.1** Publish an explicitly reviewed revision with sources, assumptions and a stable URL.
- [ ] **M5.2** Integrate lessons with Milan’s coaching website, using readable page content and appropriate navigation.
- [ ] **M5.3** Support unpublishing and replacement while keeping private drafts separate from published revisions.

**Gate:** a visitor can explore an approved lesson without accessing the studio or drafts. Later edits do not silently change the published lesson.

## M6 — Bounded visitor questions

- [ ] **M6.1** Answer questions using supported lessons and reviewed sources, with explicit handling of unsupported requests.
- [ ] **M6.2** Add public usage controls and evaluate answer quality before enabling the feature widely.

**Gate:** follow-up questions improve the explanation within its supported scope, while visitor interactions cannot modify the published original.
