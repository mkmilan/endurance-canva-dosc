# 06 — Sites creator and AI activation

Updated 4 November 2026. These steps replace the previous Atlas credential and Better Auth owner-setup procedure. The old services are retained only for migration/fallback until retirement.

## Current state

- Private Site: [Endurance Canvas](https://endurance-canvas.quirkychick26.chatgpt.site).
- Project: `appgprj_6aeb52524c808191a2e601483d5f959d`.
- Identity: native Sites sign-in; the access policy contains only Milan. No separate password, owner token or database user setup is required.
- Storage: D1 `DB` is provisioned, the usage table exists, and the production API reports it available.
- Guided generation and local studio features work. Lessons stay in the browser until exported; signing in does not upload them.
- Live AI is disabled. The reported Gateway key on Render has not been copied or verified.

## Runtime environment

| Key | Setting |
| --- | --- |
| `AI_MODE` | `demo` until the key is configured; then `live` |
| `AI_GATEWAY_API_KEY` | Existing dedicated Gateway credential, configured as a Sites secret |
| `AI_MODEL_ID` | `google/gemini-3.5-flash-lite` candidate |
| `AI_DAILY_REQUEST_LIMIT` | `20` |
| `AI_MONTHLY_REQUEST_LIMIT` | `200` |
| `AI_SITE_DAILY_REQUEST_LIMIT` | `20` |
| `AI_SITE_MONTHLY_REQUEST_LIMIT` | `200` |

Runtime changes require deployment of a saved version to take effect. Preserve the Site's audience and the D1 binding. Keep keys out of chat, Git, browser bundles, local tracked files and logs. The connected Render tool can update variables but does not expose a supported secret-read operation; the Vercel frontend has no Gateway key. Do not mutate an old service just to read its response.

## Activation and evaluation

1. Configure the existing Gateway key through supported secret settings; keep the key tied to this app and preserve any provider budget settings.
2. Set live mode and deploy the current saved version with the new environment revision.
3. Open the Site as its owner. Verify creator identity, database availability and the daily/monthly allowance.
4. Run a small real create/refine evaluation, then the prepared cases in the backend's `test/live-evaluation-cases.json` (also retained with the Site source). Record actual provider response, latency, schema outcome and explanation quality; do not label fixtures as real AI.
5. Confirm rejection of unsupported requests, invalid output, duplicate IDs and over-budget attempts while preserving the previous lesson.

The limits count attempts, including failed provider calls; they are not monetary spending caps. Default generation timeout is 25 seconds, maximum output 2,500 tokens, and paid traffic is limited to 12/minute and two active requests across the Site. The aggregate allowance prevents sharing the Site from multiplying its total request budget.

Only after draft recovery and activation/retirement decisions should the exact old resources in [the migration plan](07-sites-migration.md) be deleted. Preserve source history and avoid all other apps.
