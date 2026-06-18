# Content Pipeline

Local content command center for planning, rendering, reviewing, and analyzing short-form content across multiple app accounts.

First scope:

- He never liked me
- Tummy Time
- local dry run only
- no posting
- no image/video generation through OpenAI API

See:

- `docs/PIPELINE-SPEC.md`
- `docs/TODO.md`
- `docs/ANALYTICS-SCHEMA.md`
- `docs/DECISIONS.md`
- `docs/KLING-PROMPTING.md`
- `docs/STILL-BRIEFS.md`
- `docs/REFERENCE-PACKS.md`
- `docs/STORY-STRATEGY.md`
- `docs/CONTENT-PLANNING.md`
- `docs/POSTING-AUTOMATION.md`
- `docs/TREND-SCREENING.md`
- `docs/UGC-MAESTRO-PREVIEWS.md`
- `docs/CODEX-GOAL-PROMPT.md`

## Local Run

Start the dashboard:

```bash
pnpm dev --hostname 127.0.0.1 --port 3000
```

Open:

```text
http://127.0.0.1:3000
```

Run the full local dry run from the terminal:

```bash
pnpm pipeline:all
```

Run the first HNLM TikTok workday machine:

```bash
pnpm pipeline:hnlm-tiktok-day
```

That command scans trends, plans the HNLM TikTok day, renders local assets,
imports dummy metrics, stages publish bundles, and refreshes the advisor.
It does not publicly post. The v1 TikTok target is draft/private validation
until the TikTok audit/public-post gate is explicitly green.

Render local review videos from accepted viral hooks plus Maestro-derived app
demo segments:

```bash
pnpm pipeline:ugc-maestro-previews
```

That command keeps hook text only on the UGC/reaction part, leaves app-screen
demos clean, and keeps public posting disabled.

Run the safe background-scheduler test:

```bash
pnpm scheduler:safe-run
```

Check scheduler state:

```bash
pnpm scheduler:status
```

Install the weekday macOS LaunchAgent when you want the HNLM TikTok day to run
automatically in the background:

```bash
pnpm scheduler:install-launchd
```

See:

- `docs/BACKGROUND-SCHEDULER.md`

That run now covers:

- plan
- render
- dummy metrics
- trend scan
- publish queue preview
- advisor refresh with cached OpenAI text analysis when `OPENAI_API_KEY` is set

The dashboard buttons can also run:

- Plan Batch
- Still-Packs sync
- Render Test Assets
- Import Dummy Metrics
- Advisor aktualisieren
- Benchmark run
- Trend-Scan
- Queue pruefen
- Run Full Dry Run
- HNLM TikTok-Tag bauen
- Scheduler Safe Run
- Scheduler Status

The dashboard now also shows:

- Must-To-Dos with click-by-click detail pages
- Content Plan: when to use Reels vs Stories per account
- Reel cutdowns from existing masters
- Generation Stack: Higgsfield, OpenAI text generation, ffmpeg/Pillow rendering
- Asset detection for manual app recordings/screenshots
- Higgsfield run budget and cost-per-output efficiency
- account-aware motion-master gaps
- publish queue state with scheduled times, channel, provider, and block reason
- trend-scan output from live Google Suggest and Google Trends RSS inputs
- a plain-language "wer generiert was" explanation inside the autopilot section
- a cached advisor lane that turns KPIs and trend signals into concrete next moves
- compact on-card trend actions plus decision overrides for `auto / instrument / kill / iterate / scale / double_down`
- source-asset previews in the reusable library, including cached frames extracted from manual app videos
- technical QC for reference-pack assets, so bad ChatGPT stills or weak app assets no longer count as "ready"
- inline dashboard CSS delivery, so the cockpit does not depend on an external stylesheet chunk loading correctly in the browser

## Generation Roles

The system is intentionally split into three lanes:

- Higgsfield MCP:
  - small evergreen motion-master library only
  - gap-only generation
- ChatGPT subscription, manually:
  - still/reference generation only
  - saved into `assets/reference/*`
- local renderer:
  - reels
  - reel cutdowns
  - stories
  - carousel frames
  - reel covers
  - CTA endcards

If you need the operator-side still prompts, use:

- `docs/STILL-BRIEFS.md`

If you need to replace real app proof assets:

- open `/todos/hnlm-screen-recordings` or `/todos/tummy-time-assets`
- use the per-slot direct upload forms on the page
- or drop files into `assets/manual/<account>/` and run `python3 worker/pipeline.py manual-sync`
- same-path replacements now invalidate stale renders automatically, so refreshed assets propagate without renaming files

If you want the easiest manual still workflow:

- open the matching todo guide page and use the direct upload form
- or drop exported ChatGPT stills into `assets/inbox/reference/<account>/`
- let the page upload action or `python3 worker/pipeline.py reference-import` import them, then automatically replan and rerender affected still-based outputs
- let the importer rename and move them into `assets/reference/<account>/`

Optional advanced helper, not the main architecture:

- `worker/chatgpt_stills.mjs` still exists as a browser helper for signed-in ChatGPT work
- it is disabled by default in `config/chatgpt_stills.json`
- `pnpm pipeline:all` does not depend on it
- the standard path stays: generate stills manually in ChatGPT, then import them locally
- once stills or manual app assets exist locally, the reference lane applies a technical QC pass and only counts `passed` assets toward the pack targets

If you need the calendar and format logic, use:

- `docs/CONTENT-PLANNING.md`

Rendered review videos are written to:

```text
review/hnlm/
review/tt/
```

These folders are intentionally ignored by Git.

## Secrets

Local secrets live in:

```text
.env.local
```

`.env.local` is ignored by Git. The dashboard only shows whether OpenAI is configured; it must never print the key.

For PostHog KPI imports, this dashboard expects local variables such as:

- `POSTHOG_TT_PROJECT_ID`
- `POSTHOG_TT_PERSONAL_API_KEY`
- `POSTHOG_TT_HOST`
- `POSTHOG_HNLM_PROJECT_ID`
- `POSTHOG_HNLM_PERSONAL_API_KEY`
- `POSTHOG_HNLM_HOST`

These are for dashboard reads only. App-side PostHog SDK keys stay in the respective app repos, not here.

For Tummy Time, the pipeline can also auto-fallback to the existing local setup in:

- `/Users/marc-philliphansen/Projekte/App-Dashboard/.env`
- `/Users/marc-philliphansen/Projekte/App-Dashboard/apps.config.json`

That lets this dashboard pull live Tummy Time KPIs without duplicating the legacy keys first.

## Higgsfield

Higgsfield is now registered in Codex as a remote MCP server and authenticated via OAuth. The local pipeline defaults are:

- `kling3_0`
- `mode: std`
- `sound: off`
- `aspect_ratio: 9:16`
- `duration: 5`
- estimated cost: `7.5 credits/clip`

The local renderer still works without new Higgsfield generations by using imported clips.
Autonomous Higgsfield generation is already wired through `worker/higgsfield.py` for:

- normal gap-only motion-master runs
- benchmark pair runs
- Codex-mediated MCP polling, download, import, and QA handoff
- optional reference-image mode when the MCP surface supports it and local stills exist

The motion-master lane is now driven by:

- `config/motion_master_seeds.json`
- `config/higgsfield_benchmarks.json`
- `config/higgsfield.json` timeout knobs for longer autonomous MCP generations
- strict prompt linting before generation
- a hard per-run credit budget from `config/higgsfield.json`
- gap-only generation instead of "generate every time"
- archive-first suppression, so QA-passed existing clips can close a semantic motion gap before a new Higgsfield run is allowed
- actual next-run target selection instead of naive "missing seed" counting
- gap closure only after QA-passed masters, not merely after any downloaded clip exists
- a capped automatic retry count per seed, so failed generations do not burn credits forever
- storage-collision rejection, so reused filenames can never silently count as approved masters
- cancelled / ID-less history artifacts are rejected and never imported as valid benchmark jobs
- benchmark failures are logged separately in `data/higgsfield-benchmark-failures.json`, so cancelled challenger runs do not crash the batch and can be surfaced in the dashboard
- normal motion-run failures are also logged in `data/higgsfield-generation-failures.json`, count against retry caps, and no longer crash the whole batch

The benchmark lane is now executable, not just descriptive:

- `pnpm pipeline:higgsfield-benchmark`
- or the `Benchmark run` button in the dashboard

That run:

- reads `config/higgsfield_benchmarks.json`
- picks the highest-priority missing model/seed pairs that still fit inside the current credit cap
- respects the same per-run credit guardrails as the normal motion lane
- imports completed clips locally
- records failed benchmark attempts without treating them as valid jobs
- lets QA decide whether a pair counts toward the recommendation

It does not blindly generate every benchmark pair at once.
It only spends credits on the next missing comparison set.

Current account source policy:

- `He never liked me`: publishable reels prefer approved Higgsfield reaction masters.
- `He never liked me` v1 TikTok machine: 7 publishable TikTok assets per day
  from reusable reactions, app demos, carousels, and cutdowns; public TikTok
  autoposting remains blocked until audit/public-post readiness is green.
- `Tummy Time`: publishable reels, reel cutdowns, stories, and carousels now prefer local app images / still sources first, with archive motion left as fallback only.

Concrete generation contract:

- `HNLM`: a compact evergreen reaction-master library drives reels and cutdowns
- `Tummy Time`: manual ChatGPT stills + app proof assets drive stories, carousels, covers, CTA, and most low-cost tests
- `Higgsfield`: fills only real motion gaps
- `pipeline`: turns those reusable sources into all final outputs locally

Current evergreen library evidence:

- `He never liked me`: the reusable library now counts both QA-passed Higgsfield masters and QA-passed imported archive reactions, with explicit `archive` vs `higgsfield` provenance in the dashboard.
- `Tummy Time`: a QA sweep across the 11 legacy local motion clips rejected all 11, so Tummy Time stays intentionally still-/app-proof-first until a better motion source exists.

Current normal motion-run policy:

- next-run targets come from the approved-gap queue, not just from all missing seeds
- HNLM verdict-reaction gaps are filled first
- Tummy Time motion is parked while the still/app lane is render-ready
- seeds stop retrying after the configured attempt cap

Story and derivative policy:

- stories are not a fresh motion lane
- story stills, covers, carousel frames, and CTA plates should come from local still/app assets first
- `docs/STORY-STRATEGY.md` and `docs/CONTENT-PLANNING.md` document when each format should be used
- reference packs can now include an optional `manifest.json` so still selection is machine-guided instead of filename-only

Current research note:

- Higgsfield's current public guidance positions `Kling 3.0` as the structured scene engine and `Seedance 2.0` as a premium commercial challenger.
- Higgsfield's Jun 13, 2026 beginner workflow guide also reinforces the cheap-house rules used here: match the model to the shot goal, keep one clear camera move, and only change one variable per retry.
- This repo keeps `kling3_0` as the house default because the system is optimized for low-credit 5-second UGC masters, and the current public cost guidance is roughly `6-7` credits for Kling vs about `25` for Seedance.
- `seedance_2_0` stays in the benchmark lane as a challenger. It should only replace Kling for this repo if approved quality per credit is clearly better.
- `data/higgsfield-benchmark.json` now records that comparison state, missing benchmark pairs, the next benchmark run set, projected spend for that run, and whether a model is ready to be recommended.

Publishing is now also represented as a real local queue:

- config lives in `config/publishing.json`
- slot timing now lives in `config/schedule_strategy.json`
- scheduled items are persisted into `publish_queue` in `state.db`
- the dashboard shows why every scheduled asset is still blocked, queued, or waiting for approval
- `worker/publisher.py` writes `data/publish-execution-preview.json`, so the exact request sequence is visible before any live publish is attempted
- `pnpm pipeline:publish-carousel -- --fake-graph-test` proves the canonical Tummy Time carousel lane locally without touching a real account

Trend screening is now also automated:

- account query config lives in `config/trend_queries.json`
- `worker/trends.py` fetches live TikTok Creative Center hashtag snapshots plus Google Suggest phrases and Google Trends RSS snapshots
- raw output is written to `data/trend-inbox.json`
- top distilled pattern cards are mirrored into `config/trends.json` for the dashboard
- `data/trend-brief.json` now turns those signals into concrete next-batch experiments, source lanes, and watchouts
- the trend brief now also normalizes LLM output back into internal format enums and guarded source-lane policy
- TikTok trend rows are now account-scored, so unrelated global hashtags become format/pacing signals instead of being copied as topics

Current-Hook Intelligence is now its own layer:

- hook source config lives in `config/hook_sources.json`
- `worker/hook_intelligence.py` turns current research, creator/watchlist patterns, local winner history, and manual seeds into HNLM-specific hooks
- accepted hooks keep `inspired_by`, `source_type`, optional `source_url`, `original_pattern`, `adapted_hnlm_variant`, `adaptation_reason`, demo fit, CTA fit, safety status, and score fields
- output is written to `data/hook-intelligence.json` and timestamped reports in `data/hook-reports/`
- `pnpm hooks:safe-run` refreshes hooks without rendering, posting, or generating Higgsfield assets
- `pnpm pipeline:hook-intelligence` refreshes hooks and the dashboard
- the HNLM TikTok day flow now refreshes Hook Intelligence before planning, then uses accepted hooks in the 7-slot private/draft TikTok batch
- if live current inputs are weak, the system may use local winners/manual seeds, but the report marks fallback instead of pretending the hook is live trend evidence

Reaction Pool is now the central reusable reaction-video layer:

- pool config lives in `config/reaction_pool.json`
- `worker/reaction_pool.py` scans existing archive and imported Higgsfield reactions without copying or generating videos
- output is written to `data/reaction-pool.json` and timestamped reports in `data/reaction-pool-reports/`
- every pool asset stores `asset_id`, `file_path`, source/cost/scope fields, reusable apps, persona, emotion, intensity, vibe, age range, framing, camera style, gesture/action, best hook types, demo fit, QA status, and optional rejection reason
- manual app demo and CTA assets stay outside the app-agnostic pool and are reported as `app_specific_assets`
- `pnpm reaction-pool:safe-run` refreshes the pool without rendering, posting, or generating Higgsfield assets
- `pnpm pipeline:reaction-pool` refreshes the pool and dashboard
- the HNLM TikTok planner now selects reaction candidates from the pool via Hook Intelligence angle/emotion/demo-fit fields
- the dashboard shows pool size, approved/rejected counts, top gaps, current reaction usage, and the latest pool report
- Higgsfield remains suggest-only for missing reusable emotions/personas unless a separate explicit budget/gap decision enables generation

Demo Segment Matcher is now the HNLM-specific app-demo layer:

- demo config lives in `config/demo_segments.json`
- `worker/demo_matcher.py` scans local HNLM demo/CTA videos without copying or generating videos
- output is written to `data/demo-segments.json`, `data/demo-matcher.json`, and timestamped reports in `data/demo-matcher-reports/`
- every segment stores `segment_id`, `file_path`, app scope, demo type, promise, visual steps, duration, QA status, hook/emotion/CTA fit, and optional rejection reason
- `pnpm demo-matcher:safe-run` refreshes the matcher without rendering, posting, or generating Higgsfield assets
- `pnpm pipeline:demo-matcher` refreshes the matcher and dashboard
- HNLM `reaction_plus_cta` planning now stores matched reaction, demo segment, CTA, score, reasons, and fallback status
- HNLM `reaction_plus_cta` rendering now concatenates captioned reaction + trimmed local demo segment + matched CTA endcard
- Higgsfield stays paused until local matching and final-video quality are proven

Brief-to-Asset Generation is now the bridge from safe references to local review assets:

- `worker/brief_to_asset.py` reads `data/live-reference-capture.json` and only processes `reference_only` briefs with a clear lane
- UGC briefs render from owned Reaction Pool archive clips plus local HNLM demo segments
- Meme briefs are supported through the approved app-agnostic meme pool and place rewritten text directly on images
- output is written to `data/brief-to-asset.json`, reports in `data/brief-to-asset-reports/`, and previews in `review/brief_to_asset/`
- `pnpm brief-to-asset:safe-run` builds local previews without posting, Higgsfield generation, or ChatGPT image generation
- `pnpm pipeline:brief-to-asset` refreshes the worker and dashboard
- the dashboard shows asset plans, preview paths, missing assets, review status, and safety gates
- TikTok/Instagram remain draft/private; this stage is input for Quality Review and winner selection, not publishing
- operational details live in `docs/BRIEF-TO-ASSET.md`

Viral Reference Intelligence is now the visibility-first cloning layer:

- watchlist config lives in `config/viral_reference_watchlist.json`
- `worker/viral_reference_intelligence.py` reads competitor/live reference snapshots and creates `reference_only` winner analysis
- output is written to `data/viral-reference-intelligence.json`, timestamped reports, and local preview cards in `review/viral_reference_intelligence/`
- each post is scored for relative virality, engagement, recency, format fit, repeatability and reuse risk
- clone briefs use the lanes `pure_visibility_meme`, `pure_visibility_ugc`, `reaction_hook_clone`, `jackfriks_grid_clone`, `soft_app_bridge`, and `direct_app_demo`
- at least 80% of early briefs are visibility-first rather than direct app promotion
- the report includes sourced AI-UGC production research for reference capture, AI reactions, app usage recording, captions, composition, QA and cost control
- `pnpm viral-reference-intelligence:safe-run` runs without posting, Higgsfield generation, ChatGPT image generation, or raw creator-media reuse
- `pnpm pipeline:viral-reference-intelligence` refreshes the worker and dashboard
- operational details live in `docs/VIRAL-REFERENCE-INTELLIGENCE.md`

Posting Readiness is now the quality-gated draft/private layer:

- readiness logic lives in `worker/posting_readiness.py`
- output is written to `data/posting-readiness.json` and timestamped reports in `data/posting-readiness-reports/`
- `pnpm pipeline:posting-readiness` refreshes Hook Intelligence, Reaction Pool, Demo Matcher, plan, render, publish preview, readiness, and dashboard
- `pnpm posting-readiness:safe-run` only scans current state and writes the readiness report
- every item gets a Hook -> Reaction -> Demo -> CTA -> Render -> Queue gate matrix
- the publish queue now uses readiness blockers before an item can become draft/private-ready
- TikTok remains `draft_private` / `draft_private_hold`; public posting is still behind a closed audit/provider/credentials/AI-disclosure/dry-run/user-approval gate
- Higgsfield remains disabled for this stage; no new generation or credit spend is part of the safe command
- operational details live in `docs/POSTING-READINESS.md`

Quality Review is now the local winner/loser feedback loop:

- review logic lives in `worker/quality_review.py`
- output is written to `data/quality-review.json`, timestamped reports in `data/quality-review-reports/`, and feedback in `data/quality-feedback.json`
- manual local overrides live in `data/quality-review-overrides.json`
- `pnpm pipeline:quality-review` refreshes Posting Readiness, runs local review, writes feedback, and refreshes the dashboard
- `pnpm quality-review:safe-run` only reads current snapshots and writes review/feedback reports
- scoring checks Hook provenance, HNLM fit, Reaction fit, Demo fit, CTA fit, Render QA, duration, format, fallback status, and public-gate safety
- Hook Intelligence and Demo Matcher read `data/quality-feedback.json`, so local winners and losers influence the next batch
- TikTok remains draft/private and Higgsfield remains paused
- operational details live in `docs/QUALITY-REVIEW.md`
