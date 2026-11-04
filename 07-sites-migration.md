# Endurance Canvas — migration to ChatGPT Sites

Decision date: 4 November 2026. This plan applies only to Endurance Canvas.

## Goal

Run the current interactive learning studio, its lesson-generation API and its account/AI-usage database on ChatGPT Sites. Preserve the existing three-zone lesson, editing, review, revisions, browser drafts, source notes, JSON import/export and PNG export.

The first Sites release is private to Milan. Public lessons, website embeds, visitor questions and additional lesson types remain later product work.

## Architecture

| Existing component | Sites replacement |
| --- | --- |
| Next.js frontend on Vercel | Existing React studio in the supplied Vinext full-stack starter |
| Express backend on Render | Same-origin route handlers in the Sites Cloudflare Worker |
| MongoDB owner accounts and AI request ledger | Sites-managed sign-in and a D1 SQL request ledger |
| Better Auth owner setup/password management | Native Sites identity and the Site's owner-private access policy |
| Vercel-to-Render service token and origin/IP plumbing | Removed; API and interface share an origin |
| Browser IndexedDB drafts | Retained, with a full-library backup and restore for moving between origins |
| AI Gateway adapter | Retained server-side; real AI requires the Gateway secret in Sites |
| Three separate GitHub repositories | A single Sites source repository for the new runtime; existing repositories remain as history |

The Worker bundles the trusted lesson/source catalogues; it does not read them from a server filesystem at runtime. Native Sites identity is checked on the server before paid requests. D1 reserves attempts atomically before the provider is called; failed paid attempts remain counted. Daily/monthly caps are request limits, not a currency spending limit.

## Phase 1 — prepare

- [x] Inspect the implemented app and its API contract.
- [x] Record the exact existing infrastructure and repository revisions.
- [x] Register one new owner-private Site and persist its identity.
- [x] Choose the supported Vinext/Worker starter and D1 binding `DB`.
- [x] Finish this plan's implementation and verification record.

Baseline source: frontend `68ecb558198f287b6b51afb5a01a00c770b88d25`; backend `259760be5e75e4690765d679085933de181561ba`. Source files were available in the working copies at migration start.

## Phase 2 — adapt and build

- [x] Preserve the studio and its mobile layouts.
- [x] Replace remote proxy endpoints with `/api/status`, `/api/creator` and `/api/generate` in the Site.
- [x] Replace MongoDB/Better Auth dependencies with Sites identity and D1.
- [x] Preserve validation, unsupported-request handling, cancellation, request bounds and provider timeouts.
- [x] Preserve durable daily/monthly allowance and request-id deduplication across Worker instances.
- [x] Add complete library backup/restore, including prior revisions and conversation entries.
- [x] Make the complete backup available on the old Vercel origin.
- [ ] Configure the existing AI key in Sites through supported secret handling, without committing or printing it.

### Draft transfer

Drafts are stored in IndexedDB on the exact browser origin. A new Site cannot directly read the Vercel origin's drafts. Export the full library in each browser/device that contains drafts to keep; restore that file on Sites. A download is a backup; automatic local autosave alone is not.

The older per-lesson JSON format remains supported. It carries the current content and sources, but it does not carry the whole library, conversation or undo history. The new library format preserves those additional records after validation. Conflicting IDs and review-state treatment must be explicit in the restore flow.

No hosted owner account or cloud lesson store was activated in the old app. Atlas previously had zero database users, but this must be rechecked before any deletion. Empty Atlas setup does not establish that browsers have no saved drafts.

## Phase 3 — verify and publish

- [x] Unit/contract checks for guided generation and all supported lesson data.
- [x] D1 checks for atomic quota reservations, duplicates, concurrent requests and user isolation.
- [x] Full-library export/restore checks, including malformed files and source validation.
- [x] Production Worker build and generated schema-only D1 migration review.
- [x] Browser checks: guided create/refine, accept/reject, editing, undo, save/reopen, library and exports.
- [ ] Mobile layout and runtime-error checks where the preview environment supports them.
- [x] Publish a source-backed private Sites version and record the returned deployed URL.
- [x] Verify hosted account/database readiness without claiming a fixture is a real model call.
- [ ] Perform a bounded real AI evaluation after the secret is configured; otherwise report that gate as incomplete.

## Phase 4 — retire only this app

Retirement starts after the Sites version is verified and Milan's browser drafts are recovered or confirmed disposable. Keep the old Vercel page available until this transfer is complete. Preserve GitHub source history and this migration record.

| Service | Exact Endurance Canvas target | Scope boundary |
| --- | --- | --- |
| Vercel | Project `endurance-canva-frontend`, ID `prj_Ryn4Qhwsf7NXdoWLhTmOAFZ4M3Of`, team `team_7lKl2zSLuh9qmZQcG1Be0i0V` | Only this project and its deployments/domains |
| Render | Service `endurance-canva-backend`, ID `srv-dbl0vl6c6qcc73bm1450`, workspace `tea-cvuvbtidbo4c73f9ocdg` | Only this service and app-specific credentials |
| MongoDB Atlas | Cluster `endurance-canvas` in project `6aea1c1f1c91a0b9011d4027` | Inspect before deletion; delete only this app's dedicated resources, never the `TRAVEL` organization |

The Lactate Diary, cycling test tracker, travel/motorhome app, coaching website and shared account/workspace resources are outside this migration.

Before deletion, save non-secret configuration and identify attached resources or data. Retrieve the AI secret through supported account access or enter it directly into Sites; never rely on being able to recover it after deleting Render. Remove app-specific unused credentials only after the new app no longer depends on them.

## Rollback

Until retirement, the existing Vercel/Render deployment remains the fallback. New Sites versions have their own source history and deployment versions. Full-library JSON remains the portable draft backup across both origins. Do not repoint the old app to the new database or mix the two runtimes during verification.

## Verification record

The first private Sites version is published and its production API/database checks passed. A configured model name is not proof of real AI access.

- Site project: `appgprj_6aeb52524c808191a2e601483d5f959d`.
- Old hosting deletion: **not performed**.
- Live AI evaluation: **not performed during migration yet**.

### Verified build checkpoint

- 54 Sites tests pass: contracts, lesson state and transfer, identity/input/output gates, and SQLite concurrency/reservation behavior.
- The original frontend backup branch passes 42 tests and its Next.js production build.
- Vinext production Worker build passes. The generated schema migration creates one D1 accounting table and six indexes.
- Local D1 migration applied successfully; the actual preview `/api/status` returned `storage.available: true`, version `0.3.0-sites`, mode `demo`.
- Browser QA verified guided create/refine, accepting a suggestion, save, whole-library download and restore without overwrite, undo from restored history, manual editing, immediate account navigation and reload, native account links, and PNG export.
- A saved revision and conversation survived navigation/reload; restoring a one-lesson backup produced a separate copy alongside the original.
- PNG output inspected at 1080 × 1350. Desktop layout inspected at a 1363 px viewport without horizontal overflow. Mobile responsive rules are retained; a 360 px browser run was not available in this verification session.
- No application warnings/errors were observed in captured browser logs; browser-extension metadata errors were excluded.
- Independent source review found and fixed a restore/edit race before release.
- Evidence: [studio](assets/sites-migration/studio.jpg), [PNG export](assets/sites-migration/lesson-export.png), and [test-only full-library backup](assets/sites-migration/test-library.json). These are development QA records, not Milan's existing drafts.

### Remaining transfer gates

The connected Render tools expose environment updates but cannot read the existing AI key. The Vercel frontend contains no AI key. No key was copied, printed or committed. Guided mode is configured in Sites; live mode remains disabled until the existing key is entered through supported secret settings and real provider calls are verified.

Keep the old Vercel app available until any wanted drafts are backed up from their original browser and restored on the new Site. Only then proceed with the narrowly scoped retirement table above.

### Published version and production checks

- Live Site: [Endurance Canvas](https://endurance-canvas.quirkychick26.chatgpt.site). Owner-private; no external viewers.
- Version 1 source: `111a4463c42fde2c18733051cd471a5437db9ef5`.
- Deployment: `appgdep_6aeb56fa93d48191b4a3d14e7a90b26d`, succeeded on 4 November 2026.
- Production `DB` binding contains the migrated `generation_attempts` table.
- Authenticated service-access verification (no visitor identity supplied): `/api/status` 200 with `storage.available: true`; `/api/creator` 200 with native identity configured; guided generation 200; paid generation without a visitor identity rejected with 401 before any provider call.
- Old frontend backup support: commit `3c44420313b337e4983e83a45b24fa4dbebfb934`, production Vercel deployment `dpl_2uHDgi4R3SAK1XcaESZ4rpUUasRu`, READY.
- Real AI remains disabled because its Gateway secret has not been transferred.
- No Vercel project, Render service, Atlas cluster/project, or GitHub repository has been deleted.
