# 02 — Architecture and stack

Planning baseline: **2026-11-03**. The following choices are implementation defaults, not claims that packages or infrastructure are already installed.

## 1. Technology choices

| Area | Choice | Reason and timing |
| --- | --- | --- |
| Language | Plain JavaScript, ES modules | Fits Milan's existing workflow; runtime validation protects external data. |
| Frontend | Next.js App Router + React | Studio now; server-rendered public lesson pages later. |
| Styling | Tailwind CSS | One consistent, responsive design with visible interaction states. |
| Visuals | React + SVG; Recharts for plotted charts | A simple zone band can use SVG. Add Recharts when a real graph benefits from it. |
| Backend | Node.js + Express | Separate API repository and deployment; familiar, small request/response service. |
| AI integration | Vercel AI SDK with a server-side provider adapter | One structured generation per request; mock adapter for development. Proposed initial live route: AI Gateway. |
| Content validation | Canonical JSON Schema + application-level checks | Validate AI output, imports, and persisted content against the same contract. Ajv is the proposed runtime validator. |
| Initial storage | IndexedDB through a small storage adapter | Local drafts and revisions; explicit JSON export/import for portability. Browser storage is not a cross-device backup. |
| Later storage | MongoDB + Mongoose | Introduce for hosted, durable creator saves or publication. No database is required for the first visual/conversation prototype. |
| Verification | Vitest for meaningful logic/contract cases; Playwright for key browser flows | Boundary behaviour, failure recovery, mobile interaction, and save/reopen. |
| Hosting, later | Frontend on Vercel; Express API on Render | Proposed use of familiar services. Select plans and provision services at the hosting milestone. |
| Package management | npm; lockfiles committed in each application repo | Pin compatible current stable versions when scaffolding, including the Node runtime. |

Check version-specific documentation before writing framework or SDK calls. The official [AI SDK structured-output guide](https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data) documents schema-constrained output. [Next.js installation](https://nextjs.org/docs/app/getting-started/installation), [Tailwind's Next.js guide](https://tailwindcss.com/docs/installation/framework-guides/nextjs), and [Render's Express guide](https://render.com/docs/deploy-node-express-app) are setup references. These are implementation references, not a requirement to install dependencies during this documentation phase.

## 2. Responsibilities and request flow

The frontend owns conversation presentation, the editable preview, local storage, and file exports. The backend owns generation rules, request validation, evidence selection, and the canonical lesson contract. Later it owns durable records and published revisions.

1. The studio sends a creation or refinement request to a thin Next.js server route.
2. That route forwards it to Express. Business rules and model calls stay in Express.
3. Express supplies the selected lesson schema and a small, curated set of supported claims to the model.
4. The model returns a candidate lesson specification. Express checks its structure, parameters, registered component type, and evidence references.
5. The studio shows the validated candidate. Accepting it makes a new local draft revision and preserves the previous one for undo.
6. Moving a visual slider updates React/SVG locally. It makes no model request.

This is a bounded generation workflow. Durable agent runtimes, background queues, autonomous browsing, vector search, and code execution are outside the first-version architecture. A useful reference for connecting AI-selected data to React components is the [AI SDK generative UI guide](https://ai-sdk.dev/docs/ai-sdk-ui/generative-user-interfaces).

## 3. A lesson is data

The proposed content shape below is an **unreviewed illustrative draft**, not a published or scientifically approved lesson:

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

**Contract ownership:** at M1, create the canonical machine-readable schema and shared fixtures in the backend's `contracts/` directory. The frontend consumes a generated, read-only copy pinned to a backend commit, recording that provenance. Both repos run the same fixtures. A small sync script is sufficient; no package registry or fourth repository is needed. Generated copies are never edited by hand. This document describes the proposed contract until those files exist.

Reject unknown component types, extra executable fields, raw HTML/JavaScript, formulas supplied as code, non-finite numbers, missing units, invalid ranges, and unsupported schema versions. Require `domain.min < lt1 < lt2 < domain.max` and a default power inside the domain. Specify bounded text/array sizes in the M1 schema. Resolve approved source and claim IDs through the trusted catalogue; the model cannot introduce citation URLs.

For this example, display Z1 when `power < LT1`, Z2 when `LT1 <= power <= LT2`, and Z3 when `power > LT2`. At exact equality, also show “At LT1” or “At LT2.” This is an explicit display convention, not a claim about an instantaneous physiological switch.

Viewer exploration is temporary. Changing a slider does not change the authored default until the creator explicitly applies it. Any content/default change creates an unreviewed draft revision.

Add a later lesson type by extending the registered schema variant, renderer, approved claim cards, and shared fixtures. This keeps the conversation-based builder reusable without letting the model create arbitrary widgets. JSON imports start unreviewed; imported metadata cannot grant publication approval.

Evidence catalogue versions are immutable. Retain released versions and resolve references against the lesson's recorded `catalogVersion`, including a versioned catalogue lookup. Local drafts cache the relevant evidence metadata for offline reading; JSON exports include that snapshot for portability. An imported snapshot is display data, not trusted approval: reconcile its IDs and contents against the recorded trusted catalogue before reuse in generation or publication. If that version is unavailable, preserve the draft and explain the unresolved references instead of silently mapping them to the newest catalogue.

## 4. Proposed API

All routes below are **planned**. M1–M3 need only the first three.

| Method and route | Purpose | Stage |
| --- | --- | --- |
| `GET /health` | Minimal service health, without configuration or secrets | M1 |
| `GET /api/v1/catalog` | Supported lesson types, schema/catalog versions, trusted evidence metadata; accept `version` for an archived catalogue | M1 |
| `POST /api/v1/generations` | Create/refine a validated candidate; does not save or publish | M2 |
| `POST /api/v1/explanations` | Create a durable draft | M4 |
| `GET /api/v1/explanations` | List the creator's durable drafts | M4 |
| `GET /api/v1/explanations/:id` | Reopen a durable draft | M4 |
| `PUT /api/v1/explanations/:id` | Save with `expectedRevision`; conflict returns 409 | M4 |
| `DELETE /api/v1/explanations/:id` | Delete the owned draft and its revisions | M4 |

Generation input contains `requestId`, `mode` (`create` or `refine`), `prompt`, and, for refinement, `currentContent` plus `baseRevision`. Success returns `status: "ok"`, the request/base revision, and validated `content`. An unsupported topic returns `status: "unsupported"`, an explanation, and supported lesson types. It must not masquerade as a successful answer to another question.

Use a stable error envelope with `code`, `message`, and `requestId`; distinguish malformed requests, invalid content, throttling, provider failure, and timeout. A stale response must not replace a draft edited while the request was running. Authentication/ownership metadata is server-derived when hosted.

## 5. Configuration and hosting boundary

Proposed server-only variables include `BACKEND_API_URL` on Next.js; `AI_MODE` (`mock` initially), `AI_MODEL_ID`, and `AI_GATEWAY_API_KEY` on Express; and, later, a shared `BACKEND_SERVICE_TOKEN`, owner-session configuration, `MONGODB_URI`, and explicit usage limits. Actual example env files belong to scaffolding and contain placeholders only.

AI Gateway supports API-key authentication from external servers such as Render; see its [authentication documentation](https://vercel.com/docs/ai-gateway/authentication-and-byok). Select and evaluate a current model at M2. No model, paid account, or API key has been configured here.

Local development uses a backend bound to localhost. Before hosting creator features, authenticate the owner at the frontend and protect the separate backend as well. A thin authenticated proxy can use the server-only service token; browsers never receive it. CORS is not authentication. Secrets must not use the browser-exposed `NEXT_PUBLIC_` prefix; see [Next.js environment variables](https://nextjs.org/docs/app/guides/environment-variables).

M4 introduces persistence and access together. Use a separate project database/collections, server-assigned ownership, revision checks, and deletion of the creator's stored drafts. Keep draft content separate from the immutable approved revision used by later public pages.
