NomadGuard — AI Creative Operator

Candidate: Romeo Asis Jr. — Senior AI Engineer Applicant

An end-to-end AI creative pipeline that turns a marketing brief into a short, realistic social-media advertisement: Brief → Strategy → Creative Concept → Storyboard → Prompt Builder → Video Generation → QA → Final Asset.

The system is built around NomadGuard, a fictional travel-protection service for frequent travelers (25–40). The output is a 9:16 vertical, 10–15s, photorealistic advertisement suitable for Instagram Reels / TikTok.

Architecture

text

Marketing Brief

│

▼

Creative Planner → 3+ advertising angles (name, emotion, hook, concept, CTA)

│

▼

Concept Selector → picks one angle with rationale + audience insight

│

▼

Storyboard Generator → scene list (duration, description, camera, video\_prompt)

│

▼

Prompt Builder → model-ready prompts per scene

│

▼

VideoProvider (adapter) → generate() / getStatus() / getResult()

│

▼

Video QA → technical (ffprobe) + creative (vision model / rubric)

│

▼

Final Asset (MP4, 9:16, 10–15s)

The video layer is decoupled behind a VideoProvider interface. Only one provider is required today; adding a second (Kling, Veo, Runway, Higgsfield) means implementing the same three methods — no changes to the rest of the workflow.

Job lifecycle

Each run is treated as an asynchronous job with an explicit state machine:

text

CREATED → PLANNING → GENERATING → EVALUATING → FINALIZED

│

└── FAILED (with error, retry count, partial output)

Failed generations surface a structured error and can be retried without crashing the whole pipeline.

Setup

Prerequisites

Node.js 20+ (or Python 3.11+ if using the Python path — see below)

ffmpeg / ffprobe on PATH (for technical QA)

API key for your chosen video provider (optional — a mock provider is included)

Install

bash

git clone

cd nomadguard

npm install

Environment

Create a .env file:

bash

VIDEO\_PROVIDER=mock # or: kling | veo | runway | higgsfield

VIDEO\_API\_KEY= # required for real providers

VISION\_MODEL\_API\_KEY= # optional, for creative QA

Run

bash

npm start

This starts the local interface (see Interface below).

How to Run the Project

Open the app (default: http://localhost:3000).

The NomadGuard brief is pre-filled and editable.

Click Start Workflow.

Watch job status advance through PLANNING → GENERATING → EVALUATING → FINALIZED.

Inspect:

the three generated creative angles and the selected concept + rationale;

the storyboard (per-scene duration, camera, prompt);

the QA report (technical + creative scores);

the final MP4 for download/playback.

CLI usage (if using the CLI path):

bash

npm run cli -- --brief ./briefs/nomadguard.json --out ./output

AI / Video Providers Used

StageProvider / ToolNotes

Creative planning, storyboard, prompt buildingLLM (e.g. Claude / GPT-class)Structured JSON output, validated against schema

Video generationMockVideoProvider by default; adapter for a real provider (Kling / Veo / Runway / Higgsfield)See Provider notes below

Technical QAffprobeduration, resolution, aspect ratio, playability

Creative QAVision-capable model (optional) + rubricrealism, consistency, brief adherence, hook strength, ad quality

Provider notes

The included MockVideoProvider returns a pre-rendered placeholder MP4 so the full pipeline runs end-to-end without external credentials. The KlingProvider (or your chosen adapter) implements the same interface:

ts

interface VideoProvider {

generate(prompt: string, opts: VideoOptions): Promise<{ jobId: string }>;

getStatus(jobId: string): Promise;

getResult(jobId: string): Promise<{ url: string; meta: VideoMeta }>;

}

Production connection: swap VIDEO\_PROVIDER in .env, supply VIDEO\_API\_KEY, and the rest of the workflow is unchanged. If the real API is unavailable within the challenge window, the final video is generated manually through the provider UI and dropped into output/ — the pipeline still runs QA and finalization on it.

Important Technical Decisions

Provider abstraction first. The VideoProvider interface was defined before any provider was wired in, so mock and real providers are interchangeable.

Structured LLM output with schema validation. Every stage that calls an LLM validates the returned JSON (Zod / pydantic). Invalid output is retried with a corrective prompt rather than silently passed downstream.

Explicit job state machine. States are persisted so a failed run can be inspected and resumed; the UI reads directly from job state.

QA is a gate, not an afterthought. A video that fails technical QA (wrong duration, wrong aspect ratio, unplayable) is marked FAILED with a reason, not FINALIZED.

Creative QA is lightweight by design. Frame extraction + a rubric-driven vision check (or manual scoring fallback) — enough to demonstrate validation thinking without building an eval platform.

Trade-offs

Mock provider by default. Keeps the repo runnable with zero external setup; real-provider path is documented and adapter-ready.

Single-provider implementation. The interface supports more, but only one is wired up — matching the brief's "only one provider implementation is required."

Minimal UI. The interface is functional (edit brief, start, inspect, download) rather than polished, per Part 7 guidance.

Creative QA is rubric-based, not model-heavy. Prioritized reliability and transparency over a sophisticated evaluator.

What I Intentionally Did Not Implement

Multi-provider fan-out / A-B comparison of generated videos.

Automatic retry with prompt mutation on repeated creative QA failure (logged as a next step).

Full persistence layer (DB) — job state is stored in-process / local JSON for this exercise.

Auth, multi-tenant separation, or deployment infrastructure.

A polished frontend beyond the minimal inspect-and-run interface.

What I Would Do Next

Add a real provider adapter end-to-end (e.g. Kling or Veo) with polling, backoff, and webhook support, and verify the mock/real swap is truly drop-in.

Close the creative-QA loop: if scores fall below threshold, automatically regenerate the failing scene with a revised prompt and re-run QA — a bounded retry budget per scene.

Persist jobs (SQLite or Postgres) so runs survive restarts and can be resumed from any state.

Add a second provider and a small comparison view to validate the abstraction under real load.

Tighten the hook: add a dedicated "first 2–3 seconds" scoring pass, since hook strength dominates paid-social performance.

Tests

A small set of meaningful tests covering the highest-risk areas:

Schema validation of LLM output — malformed creative-plan or storyboard JSON is rejected and triggers a corrective retry, not a downstream crash.

Job state machine transitions — invalid transitions (e.g. CREATED → FINALIZED) are rejected; FAILED preserves the error and is retryable.

Technical QA gate — a video with the wrong aspect ratio or duration is marked invalid and does not reach FINALIZED.

Run:

bash

npm test

AI Usage

See AI\_USAGE.md for the full account of which AI tools were used, what they contributed, and what was reviewed, changed, or rejected.

Repository Layout

text

nomadguard/

├── src/

│ ├── planner/ # creative angles + concept selection

│ ├── storyboard/ # scene generation + prompt builder

│ ├── providers/ # VideoProvider interface + mock/real adapters

│ ├── qa/ # technical (ffprobe) + creative evaluation

│ ├── jobs/ # state machine + persistence

│ └── ui/ # minimal interface (or cli/)

├── tests/

├── output/ # final video + QA report

├── briefs/ # example briefs (nomadguard.json)

├── AI\_USAGE.md

└── README.md

Final Deliverable

Final video: output/nomadguard-final.mp4 (9:16, 10–15s, MP4) — included in the repo, with a downloadable link in the submission message.

QA report: output/qa-report.json (technical + creative scores).

Built as a time-boxed exercise (~4 hours active work). Incomplete areas are documented above under "What I would do next."