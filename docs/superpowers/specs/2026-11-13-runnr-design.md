# runnr technical design

Date: 2026-11-13
Status: Draft for user review. Product direction agreed; implementation choices below are proposals.

## Purpose

Help beginner runners who feel overwhelmed by conflicting advice understand their next useful step. The founder is a beginner with a soccer background who wants both a useful learning tool and a portfolio demonstration of AI engineering. The product is a mobile-friendly web app in a separate repository, deployed on Vercel and linked from the existing portfolio.

Success means a visitor can explore a convincing saved demonstration, and an eligible signed-in tester can hold a focused conversation and obtain grounded feedback on a short clip. A confident but unsupported movement claim is a failure even if the interface looks polished.

## Agreed scope

- Conversational welcome: ask what brings the runner here, then gather context as needed rather than a long compulsory questionnaire.
- Sign-in and saved conversations.
- General running education and video review, with at most 15 seconds per clip.
- Thorough analysis summarized as two or three priority observations, evidence timestamps, plain-language explanations, and practical next steps. Fewer observations are acceptable when evidence is limited.
- Follow-up questions for more detail. Report presentation is intentionally flexible.
- $0 initial operation, bounded live usage, and a saved demo that works without live inference.

Not in the first release: native app stores, training a model, medical diagnosis, injury-photo analysis, precise biomechanical measurements, tracking runs with GPS, subscriptions, or unlimited public inference. Cadence and contact timing must not be fabricated from inadequate temporal evidence.

## Approaches considered

1. Chat-first coaching: simplest, but postpones the central visual feature.
2. Coach with video review: selected by the user. Combines an accessible conversational product with a meaningful multimodal engineering showcase.
3. Pose tracking and quantitative movement analysis: useful later, but measurement validity and camera calibration significantly increase scope.

Within the selected approach, propose a managed multimodal API first. Self-hosted GPU inference is inconsistent with the initial free hosted deployment. Browser-local inference is a future alternative whose device performance and model quality would need evaluation; it is not assumed to solve the first release.

## Proposed stack

- Next.js with TypeScript, responsive UI, deployed to Vercel Hobby.
- Supabase Free for Postgres, authentication, and temporary private object storage. GitHub OAuth initially avoids reliance on production email delivery. Provider account setup remains manual until authorized/configured.
- Gemini through its server-side SDK behind a small provider adapter. Choose an available free video-capable model during the feasibility experiment; keep its identifier in configuration. Do not hardcode a model based on an assumed quota.
- Schema-validated analysis results, separate from UI rendering. Keep model/prompt/schema versions with results so examples can be reproduced.
- Static saved demonstration bundled with the application so it remains accessible when Supabase is paused or AI quota is exhausted.

The builder may choose ordinary styling libraries, component boundaries, and test tooling. Adding paid services or changing personal-media eligibility is not delegated.

## Free-tier feasibility and data boundary

Vercel Hobby supports personal noncommercial projects. Supabase free resources are adequate for a small experiment but have quotas and can pause when inactive. Neither provider is promised to remain free or always available.

Google's unpaid API terms prohibit submission of sensitive, confidential, or personal information (regional provisions vary). Consent alone is not a workaround. Thus unrestricted user videos and personal injury conversations must not be routed to this free provider by default. A warning banner or an imperfect classifier is insufficient to guarantee arbitrary chat is nonsensitive.

Proposed initial public demo: reviewed, nonsensitive example clips and scripted/general educational interactions; saved analyses require no provider request. Live free-provider experiments are restricted to developer-reviewed eligible inputs. Account identifiers stay in Supabase and are never included in model prompts. Open-ended personal coaching and arbitrary media upload are implemented only after an appropriate provider/data arrangement is selected. This is a real constraint on the full product, not a solved privacy feature.

Open decision for user review: accept this restricted free showcase initially, or investigate a local/private inference path before enabling personal uploads. No paid fallback will activate automatically.

If the restricted showcase is selected, use a fixed allowlist of reviewed prompts and example clips for visitors, with owner-only experiments for new inputs. No moderation queue is planned. The server enforces the allowlist; hiding upload controls alone is insufficient. The fuller chat/upload architecture below is staged behind this eligibility gate, not promised as enabled in the initial public demo.

Builder review completed: identified the unresolved release/input boundary above. The approval mechanism was resolved as the fixed allowlist plus owner-only experiments; the release boundary remains for user decision.

## Video flow (eligible inputs only)

1. Explain filming: fixed camera, good light, full runner in frame, preferably a side view. Ask the runner what they want reviewed.
2. Browser checks duration (up to 15 seconds), supported format, and proposed 20 MB file limit. Offer re-recording instructions for oversized/unsupported clips; transcoding is not assumed.
3. Server authenticates the user, checks feature eligibility, atomically reserves quota, and creates an upload/analysis record.
4. Issue a narrowly scoped signed upload for an unpredictable key under the user's private storage prefix. Browser uploads directly to Supabase; video bytes do not pass through a Vercel request body.
5. Server verifies ownership, actual object size/type, and independently probes duration before model submission. Evaluate a Node-compatible media metadata parser in the feasibility experiment; fail closed on unrecognized media. Client metadata is not trusted.
6. Server transfers the eligible object to the provider Files API. Persist the provider file identifier. Poll processing status through short authenticated requests rather than relying on work continuing after a response.
7. When ready, claim the analysis job atomically and request structured feedback. Bound the model call by the deployed function duration; if this cannot fit reliably, simplify the experiment before selecting additional infrastructure.
8. Validate timestamps against clip duration and validate the response schema. Display evidence-linked observations and a quality assessment; permit an explicit insufficient-evidence result.
9. Delete temporary provider and storage files after completion or terminal failure. Record deletion state and retry failed cleanup through an authenticated daily cleanup task and the next owner visit. Target deletion within 24 hours, not a guaranteed deadline. Saved results persist until the user deletes them; timestamp replay requires the original local clip or an explicitly approved demo asset.

Suggested states: awaiting_upload, validating, provider_processing, ready, analyzing, completed, failed, expired. Use unique request keys, atomic state transitions, and leases to avoid duplicate processing. Expired leases become visible retryable failures; do not silently repeat an uncertain provider request.

## Feedback contract

Store clip duration, view/quality assessment, limitations, and zero to three observations. Each observation has an evidence time range, visible description, cautious interpretation, priority, and one practical cue. Include a suggested follow-up question where it resolves uncertainty. Use plain uncertainty labels rather than invented numerical confidence scores.

Prompt instructions require distinguishing observation from inference, abstaining when the body is obscured, and not inferring injury causes. Treat messages and media text as untrusted input. A validator rejects malformed JSON and out-of-range timestamps; it cannot prove visual accuracy. Human review of representative clips is therefore an explicit release gate.

## Data and access

Minimal tables: conversations, messages, analyses, and usage_reservations. Every user-owned record has owner_id and row-level security. Analyses track state, timestamps, clip metadata, private storage key, provider identifier, schema/prompt/model versions, result, and deletion state. Avoid storing additional health profiles in this release.

Server routes authenticate every request and verify resource ownership. Storage has no public access. Service-role and AI keys are server-only. Conversation deletion also removes related analyses and schedules any remaining asset cleanup. Logs contain request IDs, durations, state transitions, and error categories, not message or media contents. Saved public examples are distinct from private user records.

## Usage controls

Starting proposals: 10 chat requests and 2 video analyses per user per day; global limits of 50 chat requests and 10 video analyses per day; one active analysis per user. These are application limits, not claims about provider allowances. Tighten them to the actual free quota, disable live inference when unavailable, and never retry quota errors automatically. Quota reservations and concurrency checks must be atomic in Postgres. Bound conversation context and generated output. No billing-enabled fallback.

## Delivery and verification

1. Feasibility experiment: verify current free-model eligibility, video timing fidelity, media probing, structured output, request duration, and cleanup with approved example clips. Record actual outcomes and costs (target $0).
2. App foundation: responsive shell, GitHub authentication, database migrations/RLS, and saved demo.
3. Chat: conversational onboarding, saved history, provider adapter, bounded context, and usage controls within the approved input boundary.
4. Video: direct uploads, validation, state transitions, structured results, follow-up context, deletion, and error states.
5. Showcase: founder-provided demo after confirming eligibility, mobile polish, deployment, README architecture notes, and portfolio link in a separate explicitly requested change.

Meaningful checks: cross-user access denied for conversations/files; invalid and overlong clips rejected server-side; concurrent requests cannot bypass limits; duplicate submissions do not duplicate jobs; provider timeouts/quota failures recover visibly; malformed output and unsupported timestamps rejected; cleanup retries work; static demo survives provider/database outage. Review clear, obscured, and unsuitable clips for unsupported claims and appropriate abstention. Report that informal evaluation is not clinical validation.

## Continuity

`HANDOFF.md` is the concise current snapshot, updated at the end of each session or meaningful milestone. `AGENTS.md` instructs future agents to read it first. Keep architecture here and later implementation tasks in a separate plan. A new chat can begin: "Continue runnr; read HANDOFF.md first." Open the runnr folder as the project so the handoff is discoverable.

## Sources checked 2026-11-13

- https://vercel.com/docs/plans/hobby
- https://vercel.com/docs/functions/limitations (4.5 MB function request/response payload limit)
- https://supabase.com/pricing
- https://supabase.com/kb/faqs/is-there-a-free-hosted-postgres-database-i-can-use-for-a-side-project
- https://ai.google.dev/gemini-api/docs/video-understanding
- https://ai.google.dev/gemini-api/docs/billing
- https://ai.google.dev/gemini-api/terms

Recheck provider terms, quotas, and runtime limits during implementation. No credentials, accounts, live tests, or deployments have been configured as part of this design.
