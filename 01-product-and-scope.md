# Endurance Canvas: product and scope

Planning date: 2026-11-03. This document describes the proposed product; implementation progress is recorded in [04-progress.md](04-progress.md).

## Purpose

Endurance Canvas helps Milan turn an endurance question into a clear, sourced, interactive explanation. He describes what he wants, sees a live preview, and refines it through conversation and simple controls.

The first release is a useful personal creation studio. Later, approved explanations can become public lessons on his coaching website. Visitor questions are a further stage, after the supported lessons work well.

These are practical starting decisions that Milan can steer as we build. Success initially means completing and reusing one good explanation; it does not depend on attracting an audience.

## The creation experience

1. Milan writes a request, such as “Explain three training zones to a beginner cyclist.”
2. The app assembles a draft with a title, short explanation, interactive visual, assumptions and source notes.
3. He explores the preview and asks for a meaningful change, such as “Make this simpler and explain what happens at LT1.”
4. He adjusts permitted text, styling and fictional example values directly when that is easier.
5. He saves the draft, reopens it later, restores the previous revision or exports it.

The interface has a conversation and an interactive preview. Both must work comfortably on a phone. Every action needs visible feedback: generating, saved, failed, restored or exported. A failed request must preserve the last useful draft.

## What the AI builds

The AI selects and configures reviewed components through a validated lesson specification. It may draft text and choose supported settings. Application code controls calculations, rendering and permitted inputs.

We begin with one component and add more when a concrete explanation requires them. Requests outside that collection should receive a clear explanation of what is supported and an appropriate next option. They should not produce an unrelated diagram presented as a successful answer.

The AI does not execute generated JavaScript, invent physiological formulas or create its own source records. A recognised source ID alone does not prove a claim: the source must actually support the explanation, and the finished lesson still needs review.

## First lesson: three training zones

This scene explains **given, fictional thresholds**. It does not calculate or diagnose an athlete’s thresholds.

| Setting | Initial teaching example |
| --- | --- |
| Metric | Cycling power in watts |
| Display range | 0–350 W |
| LT1 | 180 W |
| LT2 | 240 W |
| Initial selected power | 200 W |
| Zone convention | Z1 below LT1; Z2 from LT1 through LT2; Z3 above LT2 |

For this display, `p < 180` is Z1, `180 <= p <= 240` is Z2, and `p > 240` is Z3. At exactly either threshold, show “at LT1” or “at LT2” as well. This equality rule is a UI convention, not a claim that physiology changes abruptly at that number.

Authors can edit the fictional thresholds within validated limits. Exploring the power slider changes the preview without changing the saved starting value unless the author explicitly saves that change. Source-backed teaching text must be reviewed before the lesson can be marked ready.

## First-release boundaries

Include conversational creation and refinement, one dependable scene, manual edits, local draft persistence, revision undo, JSON import/export and one PNG export format. Use English and plain JavaScript, with mobile usability built in from the beginning.

Public membership, athlete records, activity imports, training plans, scientific forecasts, arbitrary generated code and unrestricted visitor questions belong outside this release. Website publication comes after the personal studio works.

Local development is the first environment. A hosted owner studio is optional and requires access protection and AI usage controls before exposure.

## Definition of a useful first product

Milan can request an explanation, refine it through conversation, understand and edit its assumptions, save and reopen it, and share an exported result he is happy with. Controls remain responsive without additional AI requests. The lesson clearly distinguishes fictional inputs, supported claims and material awaiting review.

See [03-features-and-build-plan.md](03-features-and-build-plan.md) for the build order and acceptance gates.
