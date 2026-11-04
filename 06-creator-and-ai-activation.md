# 06 — Creator and AI activation

Updated **2026-11-04**. This is the operational checklist for the creator-access release. Implementation, fixture verification, hosted connectivity, and real model quality are separate gates. The deployed studio must keep identifying its actual generation method.

## Current setup

| Resource | State |
| --- | --- |
| Vercel frontend | Existing `endurance-canva-frontend` project; canonical URL https://endurance-canva-frontend.vercel.app |
| Render backend | Existing free Frankfurt service `srv-dbl0vl6c6qcc73bm1450` |
| Atlas project | Created `endurance-canvas`, ID `6aea1c1f1c91a0b9011d4027` |
| Atlas cluster | Created `endurance-canvas`, **Free**, AWS `EU_CENTRAL_1`; inspected MongoDB version 8.0.36 |
| Database credential | **Not created.** Milan explicitly authorized it on 2026-11-04. The connector still refused with “User did not confirm the execution of the `atlas-create-db-user` tool so the operation was not performed.” A read-back confirmed zero database users. |
| Database network access | Milan supplied `74.220.51.0/24` and `74.220.59.0/24` from Render on 2026-11-04. Adding those exact CIDRs was refused by the connector with the same missing-confirmation error. A read-back confirmed zero access-list entries. |
| Gateway credential | Milan reports adding `AI_GATEWAY_API_KEY` directly to Render. A new manual deployment `dep-dbld74ajnfac73fllrig` is LIVE on the existing backend source. Key presence/validity and provider credits remain unverified; no live model request has run. |
| Owner account | Not created; Milan will choose his own email and password using the private setup link after database connection. |
| Actual model evaluation | **Not run.** Seven prepared cases are in the backend's `test/live-evaluation-cases.json`. |

No paid Atlas cluster, paid Render instance, model-credit purchase, or automatic credit top-up was created. The other projects' data and credentials are not used by Endurance Canvas.

## 1. Connect the isolated database

Create the app user **`endurance-canvas-app`**, scoped to cluster **`endurance-canvas`**, with only **`readWrite` on database `endurance_canvas`**. Let Atlas's credential tool generate the password. Store the resulting connection URI directly in Render's `MONGODB_URI`; never put the value in chat, source, a test artifact, or a frontend variable.

Use the new cluster's SRV connection string, append the `endurance_canvas` database, and retain TLS/default certificate verification. Encode the username/password as URI components. Keep `authSource=admin` and ordinary driver retry/write-majority options as appropriate for the Atlas connection string.

The Render outbound ranges supplied by Milan on **2026-11-04** are **`74.220.51.0/24`** and **`74.220.59.0/24`**. Add these exact two CIDRs to the new Atlas project's database IP access list. They are recorded here but are **not yet applied**. [Atlas IP access documentation](https://www.mongodb.com/docs/atlas/security/ip-access-list/)

**Connector blocker:** both `atlas-create-access-list` and `atlas-create-db-user` returned “User did not confirm…” after Milan explicitly authorized the work. This is a connector approval failure; further conversational permission is not missing. Do not retry through another tool or broaden access to get around this refusal. Until the connector can accept its own confirmation, the account owner can perform the setup in Atlas directly: open project `endurance-canvas`, add both ranges under **Database & Network Access → IP Access List**, then create the app user under **Database Users**, with the database-specific `readWrite` privilege and cluster restriction described above. Store its connection URI directly in Render. [Atlas database-user documentation](https://www.mongodb.com/docs/atlas/security-add-mongodb-users/)

Milan later reported adding the AI key and asked for another retry. Both Atlas operations were retried once under that renewed request and returned the identical confirmation refusal. Read-only verification again returned zero users and zero IP access-list entries. No database URI was generated, and the assistant did not change Render variables or trigger a deployment during those retries. The new Render manual deployment after Milan's key setup is live.

**Diagnosis:** the [official connector implementation](https://github.com/mongodb-js/mongodb-mcp-server/blob/c9ea2a51c3a9b2c10d7fcd1dbf10b10559287fa9/packages/core/src/toolBase.ts) emits this exact refusal when its separate confirmation response is false, before running the tool operation. The response does not establish that Milan declined it or explain why confirmation failed to reach the connector. Do not disable or bypass that confirmation mechanism.

Milan's screenshot now confirms that the project's **Clusters page is accessible**. The amber “Current IP Address not added” message warns about database connections from the phone's current address; it is not an Atlas dashboard lockout. Our deployed app connects from Render, so the two supplied Render CIDRs are the relevant database access entries. The [project database IP access list](https://www.mongodb.com/docs/atlas/security/ip-access-list/) is distinct from the optional organization UI list. After Milan reported changing settings, both creation operations were retried once and still failed at the connector's confirmation check. No phone-IP change is needed merely to navigate the shown Atlas page.

The server reuses one MongoDB client. Its small connection pool suits a single Render instance with two simultaneous generations and a single owner. Supported network, server-selection, socket, and pool-wait timeouts bound individual phases; application-owned queries and quota commits also have server-side time limits. Driver 7.8 no longer implements the old client `timeoutMS` option, and its transaction helper can retry for up to 120 seconds. Frontend requests have their own shorter deadline; a disconnected request must not start inference after its session check. A failed initial database connection leaves the public demo available and requires restarting/redeploying the backend after configuration is corrected. Runtime database failure blocks paid work; the app never falls back to an in-memory paid quota.

## 2. Configure creator access

| Variable | Host | Purpose |
| --- | --- | --- |
| `BACKEND_API_URL` | Vercel | Existing Render service origin |
| `BACKEND_SERVICE_TOKEN` | Both | Existing strong server-to-server credential; retain the matching value |
| `STUDIO_ORIGIN` | Both | `https://endurance-canva-frontend.vercel.app`, exactly, with no trailing slash |
| `MONGODB_URI` | Render only | Dedicated Atlas credential/connection URI |
| `MONGODB_DATABASE` | Render only | `endurance_canvas` |
| `AUTH_SECRET` | Render only | Strong random value, at least 32 characters; stable across deployments |
| `OWNER_SETUP_TOKEN_HASH` | Render only | SHA256 of a new random setup token |
| `OWNER_SETUP_EXPIRES_AT` | Render only | Explicit ISO timestamp, normally 48 hours after issuance |

A setup token is an owner-provisioning credential. Generate it only when the database is connected and the link is ready to give Milan. Do not commit its value. The private link format is `/creator#setup=<token>`; the fragment does not reach the server in the initial request, and the page removes it from the address bar. Setup sends the token in a bounded HTTPS JSON body. The server compares its hash and expiry, then invokes Better Auth's private transactional signup.

Milan chooses the account name, email, and password. The minimum password length is 12 characters. Setup does not automatically sign in; a normal sign-in verifies the credential and cookie path. The unique owner index permanently closes setup once creation commits, including concurrent requests and ambiguous network responses. A failed validation or rolled-back transaction does not consume the token.

Only the canonical production origin supports owner sessions. Preview deployments continue to offer the public demo. The Next proxy allows four Better Auth endpoints, forwards cookies only on the canonical origin, preserves all returned cookies, and removes raw session-token fields from JSON responses. Auth secrets are never exposed using `NEXT_PUBLIC_*`.

Password change requires the current password and revokes other sessions. Email delivery, email verification, and email password recovery are not configured; the email address is currently a sign-in identifier. Recovery after losing the password is an operator task, not a promised automatic email flow. Keep the chosen password in a password manager.

## 3. Connect the model

Use a dedicated Endurance Canvas **AI Gateway API key**. The current Render architecture uses external-server key authentication; a short-lived local Vercel OIDC token is not a permanent Render credential. [Gateway setup](https://vercel.com/docs/ai-gateway/getting-started) · [Key authentication](https://vercel.com/docs/ai-gateway/authentication-and-byok/api-keys)

Open the [official AI Gateway API Keys shortcut](https://vercel.com/d?title=AI+Gateway+API+Keys&to=%2F%5Bteam%5D%2F~%2Fai-gateway%2Fapi-keys), choose the Vercel team holding Endurance Canvas, and click **Create key**. Use a recognizable name such as `endurance-canvas-render`. Copy the value immediately; Vercel does not display it again.

Open the [Render backend](https://dashboard.render.com/web/srv-dbl0vl6c6qcc73bm1450), select **Environment → Add Environment Variable**, set the key to `AI_GATEWAY_API_KEY`, and paste the credential directly into its value field. Choose **Save and deploy** to activate the environment change without rebuilding code. Keep `AI_MODE=demo` until database/owner access is verified. [Render environment documentation](https://render.com/docs/configure-environment-variables)

Do not paste credentials into chat. The current connector has no Gateway-key management operation and cannot silently export its own authentication for CLI use.

| Variable | Initial value |
| --- | --- |
| `AI_MODEL_ID` | `google/gemini-3.5-flash-lite` |
| `AI_MODE` | `live` after database/owner access is verified |
| `AI_MAX_OUTPUT_TOKENS` | `2500` |
| `AI_DAILY_REQUEST_LIMIT` | `20` |
| `AI_MONTHLY_REQUEST_LIMIT` | `200` |

The model ID was selected from the complete current Gateway model catalogue. Its listed standard rates on 2026-11-03 are $0.30 per million input tokens and $2.50 per million output tokens; reasoning counts toward the output allowance. These rates can change. The model supports structured output, but this project's actual response quality and latency are still unmeasured. [Model specification](https://vercel.com/ai-gateway/models/gemini-3.5-flash-lite) · [Live catalogue](https://ai-gateway.vercel.sh/v1/models)

Before making requests, check the account's existing credit balance and eligibility. Free-credit availability depends on the account and eligible model subset; do not assume the proposed model is covered. No credit purchase or automatic top-up is part of this setup. A small dedicated key budget can provide an additional spend control; choose its amount deliberately. Gateway budgets are soft caps, so the crossing request can finish. [Pricing](https://vercel.com/docs/ai-gateway/pricing) · [Budgets](https://vercel.com/docs/ai-gateway/observability-and-spend/budgets)

The application allowance counts attempts, not dollars: 20 per UTC day and 200 per UTC month. Counts survive deployments. It reserves both periods and the request ID atomically before inference; duplicate IDs return 409. Failed/cancelled/timed-out/unsupported attempts remain counted. Sliders, manual edits, local saves, imports, exports, and the guided demo do not consume this allowance.

## 4. Verify the actual hosted flow

The [passing verification run](https://github.com/mkmilan/endurance-canva-frontend/actions/runs/41690421183) used an isolated MongoDB replica set and an explicitly labelled model fixture: 55 backend tests, 28 frontend tests, and 18 browser flows passed. The browser report is preserved in [verification/2026-11-03-creator-browser-report.json](verification/2026-11-03-creator-browser-report.json). Those tests verify account setup, cookies, sessions, quotas, and the studio interaction without spending credits. They cannot establish that Render can reach Atlas or that a real model produces useful teaching content.

After activation:

1. Verify hosted creator status, complete the private setup form, sign in, and reload the account page.
2. Confirm a signed-out browser cannot request live generation or access the owner identity/usage data. Confirm direct backend requests without the service credential return 401.
3. Sign in, explicitly select **Live AI**, and run the prepared creation and refinement cases. Confirm a valid candidate, accurate AI labelling, expected allowance changes, and preserved current content until acceptance.
4. Review every scientific statement against the fixed source catalogue. Check boundary wording, fictional-input qualifications, supported analogies, and preservation of unrequested edits. Schema success alone does not prove correctness.
5. Exercise unsupported, timeout, invalid-output, and duplicate-ID paths. The previous lesson must remain usable; failed requests must not silently switch to demo and claim an AI result.
6. Inspect the Gateway request record using its generation identifier and record the actual model, latency, token usage, and cost. Do not export secret headers or raw credentials.
7. Update Progress with the exact frontend/backend revisions and the results. Only then check off the real-model gate in M2.

## 5. What remains separate

Signing in does not upload existing lessons. Drafts still use IndexedDB and JSON backups. Cloud draft saving, public reviewed lesson URLs, website traffic integration, and visitor question answering remain later milestones. This release adds the access and operating controls needed to evaluate real AI within the existing first lesson type.
