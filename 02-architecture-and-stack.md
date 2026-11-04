# 02 — Architecture and stack

Updated: **2026-11-03**. The studio now includes creator access and a protected live-generation implementation. The public guided demo stays available. Hosted credentials and actual model evaluation remain activation gates; cloud draft storage and public publishing are later work. Current deployment and verification status belongs in [04 — Progress](04-progress.md).

## 1. Technology choices

| Area | Choice | Reason and timing |
| --- | --- | --- |
| Language | Plain JavaScript, ES modules | Fits Milan's existing workflow; runtime validation protects external data. |
| Frontend | Next.js 16.4.4 App Router + React 19.3.1 | Studio now; server-rendered public lesson pages later. |
| Styling | Tailwind CSS 4.3.4 | One consistent, responsive design with visible interaction states. |
| Visuals | React + SVG | The first scene uses a responsive SVG zone diagram. Recharts remains an option when later plotted charts justify it; it is not installed. |
| Backend | Node.js 24.22.1 + Express 5.3.0 | Separate API repository and deployment; familiar request/response service. Both repos target Node 24. |
| AI integration | Vercel AI SDK 7.0.162 with a server-side AI Gateway adapter | Every request explicitly selects guided demo or live AI. Live requires the owner session, persistent allowance, and configured credentials. Actual model evaluation is still pending. |
| Creator access | Better Auth 1.7.15 + MongoDB adapter 1.7.15 | One owner, email/password sign-in, HttpOnly sessions, no public signup and no external authentication subscription. |
| Content validation | Canonical JSON Schema + dependency-free semantic validator | The shared validator runs in both applications. Backend tests use Ajv 8.20.0 to check schema compatibility and parity cases. |
| Initial storage | IndexedDB through a small storage adapter | Local drafts and revisions; explicit JSON export/import for portability. Browser storage is not a cross-device backup. |
| Owner/usage storage | MongoDB Atlas Free + native driver 7.8.0 | Separate Endurance Canvas cluster for sessions, the owner credential, rate limits, and transactional AI allowances. No Mongoose layer is needed for this small model. |
| Later lesson storage | MongoDB | Cloud drafts and published revisions are separate future features. Creating the auth database does not make local lessons sync across devices. |
| Verification | Node's built-in `node:test`; browser checks for the user flow | Backend tests cover boundaries, contracts, protected routes, and failure handling. Record browser and deployed-flow evidence separately in Progress. |
| Testing hosting | Frontend on Vercel; Express API on Render's free instance | Milan selected these hosting platforms for testing. The Vercel project is connected; Render has a dedicated backend service. |
| Package management | npm with exact dependency versions and lockfiles | The implementation pins its packages and Node runtime. Update versions deliberately and verify the resulting builds. |

Check the installed version's documentation before changing framework or SDK calls. The live adapter was written against the installed AI SDK's bundled structured-output documentation; this does not replace a real-model evaluation. Further references are the [AI SDK structured-output guide](https://ai-sdk.dev/docs/ai-sdk-core/generating-structured-data), [Next.js installation](https://nextjs.org/docs/app/getting-started/installation), [Tailwind's Next.js guide](https://tailwindcss.com/docs/installation/framework-guides/nextjs), and [Render's Express guide](https://render.com/docs/deploy-node-express-app).

## 2. Responsibilities and request flow

The frontend owns conversation presentation, the editable preview, local storage, and file exports. The backend owns owner sessions, AI allowances, generation rules, evidence selection, and the canonical lesson contract. Later it owns durable drafts and published revisions.

1. The studio sends a creation or refinement request to a thin Next.js server route.
2. That route forwards it to Express. Business rules and model calls stay in Express.
3. Guided demo uses prepared content without a model call. Live AI verifies the owner session in MongoDB, then reserves the request ID and daily/monthly allowance in a transaction before contacting AI Gateway.
4. The live adapter requests structured data using the same evidence catalogue. Both paths check structure, parameters, registered component type, and evidence references. Unsupported questions and invalid output preserve the current lesson.
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

Owner routes and generation are implemented. Durable-save routes remain planned and are not available in the current API.

| Method and route | Purpose | Status |
| --- | --- | --- |
| `GET /health` | Public health, service version, and `demo`/`live` mode; no secrets | Implemented |
| `GET /api/v1/catalog` | Protected catalogue/schema response; optional `version=1` | Implemented |
| `POST /api/v1/generations` | Create/refine candidate; live additionally requires an owner session and quota reservation | Implemented; actual model evaluation pending |
| `GET /api/v1/creator` | Session and AI availability; usage counts only for the signed-in owner | Implemented |
| `POST /api/v1/creator/setup` | One-time, expiring-token owner setup | Implemented; closes permanently once the owner exists |
| `POST /api/auth/sign-in/email` | Password sign-in and HttpOnly session cookie | Implemented |
| `POST /api/auth/sign-out` | Revoke the session and clear its cookie | Implemented |
| `POST /api/auth/change-password` | Change password with the current password | Implemented |
| `GET /api/auth/get-session` | Read/renew the authenticated session | Implemented |
| `POST /api/v1/explanations` | Create a durable draft | Planned M4 |
| `GET /api/v1/explanations` | List the creator's durable drafts | Planned M4 |
| `GET /api/v1/explanations/:id` | Reopen a durable draft | Planned M4 |
| `PUT /api/v1/explanations/:id` | Save with `expectedRevision`; conflict returns 409 | Planned M4 |
| `DELETE /api/v1/explanations/:id` | Delete the owned draft and its revisions | Planned M4 |

Generation input contains `mode` (`create` or `refine`), `generationMode` (`demo` or `live`, default `demo`), `prompt`, an optional `requestId`, and, for refinement, `currentContent` plus an optional `baseRevision`. Success returns `status: "ok"`, request/base revision, response `mode`, validated `content`, and a message identifying the generation method. An unsupported topic returns `status: "unsupported"`, an explanation, and supported lesson types. Reusing a live request ID returns 409 without another provider call, including after a process restart.

Errors use `status: "error"` with an `error` object containing `code`, `message`, and `requestId`. The API distinguishes malformed requests, invalid content, throttling, provider failure, and timeout. The frontend checks request IDs and base revisions before applying candidates. Durable ownership metadata will be server-derived when cloud storage is introduced.

## 5. Configuration and hosting boundary

Next.js holds `BACKEND_API_URL`, `BACKEND_SERVICE_TOKEN`, and the canonical `STUDIO_ORIGIN`. Express additionally accepts `MONGODB_URI`, `MONGODB_DATABASE`, `AUTH_SECRET`, the hashed expiring owner-setup token, `AI_MODE`, `AI_MODEL_ID`, `AI_GATEWAY_API_KEY`, and request limits. The repositories contain safe example env files; secret values belong only in hosting settings. See [06 — Creator and AI activation](06-creator-and-ai-activation.md) for the exact operational sequence and current blockers.

AI Gateway supports API-key authentication from external servers such as Render; see its [authentication documentation](https://vercel.com/docs/ai-gateway/authentication-and-byok). The current evaluation candidate is `google/gemini-3.5-flash-lite`, verified in the live model catalogue on 2026-11-03. A model ID is not evidence that the account has credits or that its output quality has passed. The app fails closed for live generation whenever owner storage or provider configuration is unavailable.

Local backend development binds to localhost. Production binds to `0.0.0.0` and Render's `PORT`; startup requires a service token of at least 32 characters. All `/api` endpoints verify that token, and the Next proxy replaces incoming authorization with its own credential. Vercel's default deployment protection is retained separately from application owner access. Secrets must not use the browser-exposed `NEXT_PUBLIC_` prefix; see [Next.js environment variables](https://nextjs.org/docs/app/guides/environment-variables).

The Render test backend is at [endurance-canva-backend.onrender.com](https://endurance-canva-backend.onrender.com). Current deployment revisions and checks belong in [Progress](04-progress.md). A dedicated Atlas Free cluster now exists in AWS Frankfurt. The Atlas connector refused database-user and access-list creation despite Milan's explicit authorization on 2026-11-04, so hosted owner storage is not yet connected. See Activation for the recorded Render CIDRs and manual setup path.

## 6. Owner sessions and persistent allowances

Better Auth runs on Render, backed by MongoDB. The browser calls same-origin Next routes; the proxy forwards cookies and every `Set-Cookie` header. The backend trusts one configured frontend origin. Unused auth endpoints, including public signup, are blocked in both services. Passwords are handled by Better Auth, with a 12-character minimum. Production session cookies are host-only, HttpOnly, Secure, and SameSite=Lax; paid requests verify the session in the database without a cookie cache.

Setup uses a private signup instance that is never exposed over HTTP. A random expiring setup token is hashed in configuration and delivered through a URL fragment, which the page immediately removes. The MongoDB unique owner index and transactional signup allow one owner even under concurrent requests. Setup failure before commit permits a retry; an existing owner closes setup. Email is a sign-in identifier; email verification and email recovery are not configured.

The default live allowance is **20 attempts per UTC day and 200 per UTC month**. A transaction reserves both counters and a unique owner/request ID before generation. Failed, timed-out, cancelled, and unsupported provider attempts remain counted because they may incur cost. These are request allowances, not exact monetary caps. A separate Gateway key budget can add a provider-side spend control when that credential is created.

Usage records contain request IDs, a hash of inputs, timestamps, status, and available token/provider identifiers. They do not store raw prompts, generated lessons, or passwords. Request identifiers remain recorded for idempotency. Auth records and usage persist across Render restarts; local lessons remain in IndexedDB.

References: [Better Auth Express integration](https://better-auth.com/docs/integrations/express), [MongoDB adapter and transactions](https://better-auth.com/docs/adapters/mongo), [cookies](https://better-auth.com/docs/concepts/cookies), and [rate limits](https://better-auth.com/docs/concepts/rate-limit).
