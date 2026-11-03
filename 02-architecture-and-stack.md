# 02 — Architecture and stack

First implementation baseline: **2026-11-03**. The initial studio and backend are implemented for testing. Guided demo generation is available in code; real-model evaluation, cloud draft storage, and public lesson publishing remain future work. Current deployment and verification status belongs in [04 — Progress](04-progress.md).

## 1. Technology choices

| Area | Choice | Reason and timing |
| --- | --- | --- |
| Language | Plain JavaScript, ES modules | Fits Milan's existing workflow; runtime validation protects external data. |
| Frontend | Next.js 16.4.4 App Router + React 19.3.1 | Studio now; server-rendered public lesson pages later. |
| Styling | Tailwind CSS 4.3.4 | One consistent, responsive design with visible interaction states. |
| Visuals | React + SVG | The first scene uses a responsive SVG zone diagram. Recharts remains an option when later plotted charts justify it; it is not installed. |
| Backend | Node.js 24.22.1 + Express 5.3.0 | Separate API repository and deployment; familiar request/response service. Both repos target Node 24. |
| AI integration | Vercel AI SDK 7.0.162 with a server-side adapter | `AI_MODE=demo` uses prepared teaching content and editing rules. An optional AI Gateway live adapter exists but is unconfigured and unevaluated. |
| Content validation | Canonical JSON Schema + dependency-free semantic validator | The shared validator runs in both applications. Backend tests use Ajv 8.20.0 to check schema compatibility and parity cases. |
| Initial storage | IndexedDB through a small storage adapter | Local drafts and revisions; explicit JSON export/import for portability. Browser storage is not a cross-device backup. |
| Later storage | MongoDB + Mongoose | Introduce for hosted, durable creator saves or publication. No database is required for the first visual/conversation prototype. |
| Verification | Node's built-in `node:test`; browser checks for the user flow | Backend tests cover boundaries, contracts, protected routes, and failure handling. Record browser and deployed-flow evidence separately in Progress. |
| Testing hosting | Frontend on Vercel; Express API on Render's free instance | Milan selected these hosting platforms for testing. The Vercel project is connected; Render has a dedicated backend service. |
| Package management | npm with exact dependency versions and lockfiles | The implementation pins its packages and Node runtime. Update versions deliberately and verify the resulting builds. |

Check the installed version's documentation before changing framework or SDK calls. The live adapter was written against the installed AI SDK's bundled structured-output documentation; this does not replace a real-model evaluation. Further references are the [AI SDK structured-output guide](https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data), [Next.js installation](https://nextjs.org/docs/app/getting-started/installation), [Tailwind's Next.js guide](https://tailwindcss.com/docs/installation/framework-guides/nextjs), and [Render's Express guide](https://render.com/docs/deploy-node-express-app).

## 2. Responsibilities and request flow

The frontend owns conversation presentation, the editable preview, local storage, and file exports. The backend owns generation rules, request validation, evidence selection, and the canonical lesson contract. Later it owns durable records and published revisions.

1. The studio sends a creation or refinement request to a thin Next.js server route.
2. That route forwards it to Express. Business rules and model calls stay in Express.
3. In the testing release, Express's guided demo adapter creates or revises a lesson using prepared content and supported editing rules. It makes no model call and labels its response accordingly.
4. Express checks the candidate's structure, parameters, registered component type, and evidence references. The optional live adapter would instead request schema-constrained model output using the same evidence catalogue; the frontend currently blocks that live mode.
5. The studio shows the validated candidate. Accepting it makes a new local draft revision and preserves the previous one for undo.
6. Moving a visual slider updates React/SVG locally. It makes no model request.

This is a bounded generation workflow. Durable agent runtimes, background queues, autonomous browsing, vector search, and code execution are outside the first-version architecture. A useful reference for connecting AI-selected data to React components is the [AI SDK generative UI guide](https://ai-sdk.dev/docs/ai-sdk-ui/generative-user-interfaces).

## 3. A lesson is data

The implemented content contract accepts the following **unreviewed illustrative draft**. It is not a published or scientifically approved lesson:

```json
{
  "schemaVersion": 1,
  "catalogVersion": 1,
  "lessonType": "three-zone-explainer",
  "title": "Explore three training zones",
  "audience": "beginner",
  "blocks": [
    {
      "type": "paragraph",
      "text": "Move the power control to explore this fictional example.",
      "claimIds": []
    }
  ],
  "visual": {
    "component": "three-zone-power",
    "unit": "W",
    "domain": { "min": 0, "max": 350 },
    "lt1": 180,
    "lt2": 240,
    "defaultPower": 200
  },
  "presentation": { "theme": "light", "accent": "teal" },
  "sourceIds": [],
  "assumptions": [
    "Thresholds are fictional inputs supplied for teaching.",
    "Changing a threshold changes the example; it does not predict an adaptation."
  ]
}
```

The application adds identity, revision, timestamps, and review state outside this content object. Local storage owns that metadata initially; the server becomes authoritative for durable records later. The model cannot grant approval, choose an owner, or publish.

**Contract ownership:** the backend's `contracts/` directory contains `lesson.schema.json`, `catalog.v1.json`, `example.v1.json`, and `validation.js`. The frontend's `npm run sync:contracts` copies these into `src/generated/contracts/` and records the backend commit and file hashes in `provenance.json`. Sync from the published, clean backend revision before a frontend release. Generated copies are never edited by hand. Shared fixture checks keep the two applications aligned.

The validator rejects unknown component types, extra executable fields, raw HTML/JavaScript, formulas supplied as code, non-finite numbers, missing units, invalid ranges, and unsupported schema versions. It requires `domain.min < lt1 < lt2 < domain.max` and a default power inside the domain. The current range starts at 0 W and ends between 100 and 1000 W. Title, paragraph, and array sizes are bounded. Source and claim IDs resolve through the trusted catalogue; the model cannot introduce citation URLs.

For this example, display Z1 when `power < LT1`, Z2 when `LT1 <= power <= LT2`, and Z3 when `power > LT2`. At exact equality, also show “At LT1” or “At LT2.” This is an explicit display convention, not a claim about an instantaneous physiological switch.

Viewer exploration is temporary. Changing a slider does not change the authored default until the creator explicitly applies it. Any content/default change creates an unreviewed draft revision.

Add a later lesson type by extending the registered schema variant, renderer, approved claim cards, and shared fixtures. This keeps the conversation-based builder reusable without letting the model create arbitrary widgets. JSON imports start unreviewed; imported metadata cannot grant publication approval.

Evidence catalogue versions are immutable. Version 1 is implemented; an unavailable version returns an explicit error. Preserve released catalogues when introducing later versions. Local drafts and JSON exports retain the relevant evidence snapshot for portability. An imported snapshot is display data, not trusted approval: validate references against the recorded trusted catalogue before reuse in generation or publication. Unsupported imports must leave the current lesson unchanged rather than silently mapping evidence to a newer catalogue.

## 4. API: implemented routes and later extensions

The first three routes are implemented. Durable-save routes remain planned and are not available in the current API.

| Method and route | Purpose | Status |
| --- | --- | --- |
| `GET /health` | Public health, service version, and `demo`/`live` mode; no secrets | Implemented |
| `GET /api/v1/catalog` | Protected catalogue/schema response; optional `version=1` | Implemented |
| `POST /api/v1/generations` | Protected create/refine candidate; does not save or publish | Implemented with guided demo; live adapter not evaluated |
| `POST /api/v1/explanations` | Create a durable draft | Planned M4 |
| `GET /api/v1/explanations` | List the creator's durable drafts | Planned M4 |
| `GET /api/v1/explanations/:id` | Reopen a durable draft | Planned M4 |
| `PUT /api/v1/explanations/:id` | Save with `expectedRevision`; conflict returns 409 | Planned M4 |
| `DELETE /api/v1/explanations/:id` | Delete the owned draft and its revisions | Planned M4 |

Generation input contains `mode` (`create` or `refine`), `prompt`, an optional `requestId`, and, for refinement, `currentContent` plus an optional `baseRevision`. Success returns `status: "ok"`, request/base revision, response `mode`, validated `content`, and a message identifying guided demo use. An unsupported topic returns `status: "unsupported"`, an explanation, and supported lesson types. It does not masquerade as a successful answer to another question.

Errors use `status: "error"` with an `error` object containing `code`, `message`, and `requestId`. The API distinguishes malformed requests, invalid content, throttling, provider failure, and timeout. The frontend checks request IDs and base revisions before applying candidates. Durable ownership metadata will be server-derived when cloud storage is introduced.

## 5. Configuration and hosting boundary

Current server-only variables include `BACKEND_API_URL` and `BACKEND_SERVICE_TOKEN` on Next.js, and `AI_MODE=demo`, `BACKEND_SERVICE_TOKEN`, and generation limits on Express. `AI_MODEL_ID` and `AI_GATEWAY_API_KEY` are optional live-adapter settings and are not configured. Safe example env files are included. Owner-session configuration and `MONGODB_URI` remain later work.

AI Gateway supports API-key authentication from external servers such as Render; see its [authentication documentation](https://vercel.com/docs/ai-gateway/authentication-and-byok). Select and evaluate a current model before completing the live-AI milestone. The testing studio explicitly refuses live generation until owner-access protection is implemented.

Local backend development binds to localhost. Production binds to `0.0.0.0` and Render's `PORT`; startup requires a service token of at least 32 characters. Both catalogue and generation endpoints verify that token, and the frontend proxy keeps it server-side. Vercel's default SSO/deployment-protection configuration is retained; this is not a substitute for the planned application owner access required before live AI. Secrets must not use the browser-exposed `NEXT_PUBLIC_` prefix; see [Next.js environment variables](https://nextjs.org/docs/app/guides/environment-variables).

The Render test backend is live at [endurance-canva-backend.onrender.com](https://endurance-canva-backend.onrender.com), running source revision [5790a29](https://github.com/mkmilan/endurance-canva-backend/commit/5790a2901212591a293313984f81058133cf8d17). Deployment checks confirmed `/health` returns HTTP 200 in demo mode, and unauthenticated catalogue and generation requests return HTTP 401. These checks establish backend health and its access boundary; frontend deployment and end-to-end behaviour are recorded separately in [Progress](04-progress.md). Atlas access has been confirmed, but no project database has been provisioned or integrated for this app.

M4 introduces persistence and access together. Use a separate project database/collections, server-assigned ownership, revision checks, and deletion of the creator's stored drafts. Keep draft content separate from the immutable approved revision used by later public pages.
