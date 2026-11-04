# 04 — Progress

Updated: **4 November 2026**.

## Current position

**Endurance Canvas is built and published privately on [ChatGPT Sites](https://endurance-canvas.quirkychick26.chatgpt.site).** The existing guided builder, interactive three-zone lesson, preview/accept/reject flow, editing, review, revisions, browser library and exports are preserved. New full-library backup/restore makes migration between browser origins possible without losing history or conversation.

Sites now runs the frontend and API together, supplies sign-in, and provisions D1 for durable AI accounting. The previous Atlas credential/setup problem is no longer a dependency of the new app.

**Real AI is still pending.** The existing Gateway secret is in Render and cannot be read by the connected Render tool. It must be securely added to Sites before enabling live mode and running the model evaluation. No real provider call or purchase occurred during migration.

## Release record

| Resource | Verified state |
| --- | --- |
| New Site | `appgprj_6aeb52524c808191a2e601483d5f959d`, version 1, owner-private |
| Source | `111a4463c42fde2c18733051cd471a5437db9ef5` in the Site source repository |
| Deployment | `appgdep_6aeb56fa93d48191b4a3d14e7a90b26d`, succeeded |
| Production database | D1 binding `DB`, table `generation_attempts` |
| Old Vercel backup update | Frontend commit `3c44420313b337e4983e83a45b24fa4dbebfb934`; deployment `dpl_2uHDgi4R3SAK1XcaESZ4rpUUasRu`, READY |
| Old Render backend | Existing service `srv-dbl0vl6c6qcc73bm1450` retained |
| Old Atlas resources | Project `6aea1c1f1c91a0b9011d4027`, cluster `endurance-canvas` retained |

## Verification

- **54 Sites tests pass**, covering schema/claims parity, lesson state, full-library transfer, native identity gates, request bounds, safe provider errors, cancellation, timeouts, quotas and duplicate IDs. Concurrent independent SQLite writers exercise the actual generated reservation SQL.
- **42 original frontend tests pass** for the narrowly scoped library backup update. Its production Next.js build passes and its Vercel deployment is READY.
- **Production Worker build passes.** The schema-only D1 migration was generated, reviewed, tested locally and applied by Sites publication.
- **Production HTTP checks using existing service access:** status 200 and database available; creator 200 and identity configured; guided generation 200; identity-less paid generation rejected with 401 before a provider call. Service access supplies no visitor identity, so this is not a signed-in-owner browser test.
- **Browser QA:** guided create/refine, accept, save, full-library backup and restore without overwriting the original, undo from restored history, edit, account navigation, reload persistence and PNG export. Export inspected at 1080 × 1350; desktop had no horizontal overflow. Existing mobile rules are retained; a fresh 360 px browser run was not performed.
- No application warnings/errors were observed in captured browser logs. Browser-extension metadata errors were excluded. Independent review identified and fixed a restore/edit race before release.

Evidence: [verification record](verification/2026-11-04-sites-migration.json), [studio](assets/sites-migration/studio.jpg), [PNG](assets/sites-migration/lesson-export.png). These use test data, not Milan's drafts.

## Remaining work, in order

1. Try the new private Site and transfer any wanted browser drafts: old app **My lessons → Back up library**, new app **My lessons → Restore library**.
2. Configure `AI_GATEWAY_API_KEY` in Sites, enable live mode, and run the prepared real-model evaluation. Keep schema/fixture success separate from actual model quality and provider access.
3. Retire only the Endurance Canvas Vercel project, Render service and dedicated Atlas resources after those transfer gates. No deletion has happened.
4. Resume the product roadmap: cloud lesson synchronization only if needed, then reviewed public lessons/website integration and bounded visitor questions.

[Migration plan](07-sites-migration.md) records exact cleanup targets and rollback. Previous release details and CI evidence remain in Git history and the older verification files.
