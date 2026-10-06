# SynCare — Risk-Adaptive Post-Discharge Monitoring Workflow (Hacktiv8 x IBM 2026)

## 1. Executive Summary & System Scope

**SynCare** is a risk-adaptive post-discharge patient monitoring prototype tailored for the Indonesian healthcare context, developed for the **Hacktiv8 x IBM 2026** initiative. The planned workflow targets post-discharge care; the current synthetic fixture covers heart failure (`discharge-chf-high.txt`). The system coordinates care protocols by dividing responsibilities between conversational engagement and structured workflow orchestration.

### Architectural Division (Planned Architecture)
* **IBM Bob (Conversational Front-End)**: Manages patient and caregiver dialogue, collects daily symptom reports, and provides empathetic interactive communication.
* **Langflow (Backend Workflow Orchestration)**: Handles structured data extraction, deterministic heuristic risk tiering, care plan drafting, human-in-the-loop clinical review gates, and grounded guideline retrieval.

> **Current Implementation Status**: **Five separate flows have passed synthetic fixture acceptance; the planned architecture is NOT a demonstrated integrated end-to-end pipeline and is NOT production or clinically ready.** **F1–F4 PASSED** synthetic acceptance on both the live Linux host (`http://127.0.0.1:7860`) and historical Windows machine through **STOP 2** (F1 yields 7/HIGH with complete episode; F2 proves tier adjustment and structural red-flag bypass; F3 yields a guarded Indonesian `satuseshat` draft and fail-closed missing facts; F4 proves actual human pause/resume, approve-only SQLite persistence, escalation, and material nonclinical critique-driven redrafting). Review database created at `/home/frxskie/langflow/syncare-review.sqlite3` with exactly 1 approved plan and 3 audit logs (stored records verified unchanged across all F5 acceptance runs). Flow F5 was authorized via `checkpoint-2-human-gate selesai, lanjut` and created as `4fff7e76-d710-4662-9b8e-087b5182a245` (11 nodes, 10 edges post-T4 single-edge fix). F5 is **ACCEPTED** (2026-10-05): after the operator updated the global Gemini credential (agent never inspected or changed key values), the unchanged flow code/model settings passed the live RAG route — in-scope diet keycheck with citations 13:38 UTC, out-of-scope cancer and out-of-plan insulin exact refusals 14:26 UTC (full trace evidence in [`reports/answer_care_questions-test.txt`](reports/answer_care_questions-test.txt) Section 10). Fresh official all-5 exports, `notify_done` (status ok), and the export/source audit are complete; **STOP 3 COMPLETE — halt and await further operator instructions; no extra flows**.

### Clinical Disclaimer & Synthetic Data Mandate
* **SYNTHETIC DEMO ONLY**: All clinical data, patient profiles, and workflow artifacts are synthetic demonstrations.
* **NOT CLINICALLY APPROVED**: This workflow is an engineering prototype and has not received clinical review or authorization from licensed medical practitioners.
* **NO MEDICAL DIAGNOSIS / NO DOSAGE ADJUSTMENTS**: SynCare does not diagnose medical conditions, does not modify medications or dosages, and does not provide autonomous emergency medical advice.
* **MANDATORY HUMAN OVERSIGHT**: All clinical escalation paths, care plans, and protocol adaptations require formal review and approval by human medical staff.
* **NO STANDALONE APP / NO TEST SUITE**: This repository contains no standalone web/mobile application, no package dependency manifest (`package.json`, `requirements.txt`), and no automated test suite. The repo consists strictly of Langflow flow graphs, text fixtures, educational protocol drafts, and verification audit logs.

---

## 2. Repository Directory Structure & Artifact Links

```text
.
├── exports/
│   ├── register_discharge_summary.json       # Verbatim redacted F1 export (4 nodes, 3 edges)
│   ├── record_patient_checkin.json           # Verbatim redacted F2 export (7 nodes, 6 edges)
│   ├── draft_followup_plan.json              # Verbatim redacted F3 export (6 nodes, 6 edges)
│   ├── review_care_plan.json                 # Verbatim F4 export (12 nodes, 11 edges)
│   └── answer_care_questions.json            # Fresh official F5 export, ACCEPTED 2026-10-05 (11 nodes, 10 edges; api_key refs masked by export tool)
├── fixtures/
│   ├── discharge-chf-high.txt                # Synthetic Indonesian CHF discharge summary text
│   ├── f3-plan-input-chf-high.json           # Actual F1 JSON plus synthetic supplied scheduling facts
│   └── f3-plan-input-missing-facts.json      # Same patient, empty medications, absent date/contact
├── protocols/
│   ├── hf-aftercare.txt                      # Generic CHF post-discharge educational draft (429 words)
│   └── medication-safety.txt                 # Generic medication adherence and safety draft (391 words)
├── reports/
│   ├── langflow-discovery.txt                # Langflow MCP environment & component discovery report
│   ├── langflow-discovery-transcript.txt     # Raw and summarized tool schemas & discovery transcript
│   ├── register_discharge_summary-test.txt   # F1 history and actual passed fixture/session evidence
│   ├── record_patient_checkin-test.txt       # F2 full actual transcript, three runs, bypass proof
│   ├── draft_followup_plan-test.txt          # F3 actual transcript, two isolated runs, guardrail proof
│   ├── review_care_plan-test.txt             # F4 pause/resume, per-run SQLite rows/counts, critique revision
│   └── answer_care_questions-test.txt        # F5 audit, 429 quota blocker, native bypass, verifier tests
└── README.md                                 # Full agent handoff documentation
```

### Educational Protocol Drafts & Audit Artifacts
* [`protocols/hf-aftercare.txt`](protocols/hf-aftercare.txt) (429 words): Generic Indonesian post-discharge education draft covering activity guidelines, sodium restriction, daily weight tracking, and hospital contact. Strictly pending clinician review.
* [`protocols/medication-safety.txt`](protocols/medication-safety.txt) (391 words): Generic Indonesian medication adherence draft emphasizing schedule consistency, safe storage, and strict prohibition of unapproved dosage changes. Strictly pending clinician review.
* [`reports/langflow-discovery.txt`](reports/langflow-discovery.txt): Environment discovery report documenting connected Langflow MCP tools, component registry, and inventory of 35 existing server flows.
* [`reports/langflow-discovery-transcript.txt`](reports/langflow-discovery-transcript.txt): Tool schema extractions and discovery transcript.
* [`reports/register_discharge_summary-test.txt`](reports/register_discharge_summary-test.txt): F1 repair history, verbatim mutation/validation results, gemini-2.5-flash 404 blocker, and PASSED gemini-3.5-flash-lite fixture run (7/HIGH) with MCP vs REST session evidence.
* [`exports/register_discharge_summary.json`](exports/register_discharge_summary.json): Exact raw export of F1 flow graph (nodes, edges, config, and source).
* [`reports/record_patient_checkin-test.txt`](reports/record_patient_checkin-test.txt): Full actual F2 construction/registration transcript, isolated REST tests, exact outputs, and scorer-bypass evidence.
* [`exports/record_patient_checkin.json`](exports/record_patient_checkin.json): Exact raw redacted F2 flow export.
* [`reports/draft_followup_plan-test.txt`](reports/draft_followup_plan-test.txt): Actual F3 mutations, validation, high/missing-facts outputs, source-input provenance, and session/guardrail evidence.
* [`exports/draft_followup_plan.json`](exports/draft_followup_plan.json): Exact unmodified MCP export of the six-node guarded drafting flow.
* [`reports/review_care_plan-test.txt`](reports/review_care_plan-test.txt): Actual F4 pause/resume streams, all three human actions, material critique response, initial failed-run evidence, and untruncated read-only SQLite row/count queries.
* [`exports/review_care_plan.json`](exports/review_care_plan.json): Exact unmodified MCP export of the twelve-node human review flow. The SQLite database remains outside the repository and is not committed.
* [`reports/answer_care_questions-test.txt`](reports/answer_care_questions-test.txt): F5 acceptance report — live RAG proof (diet keycheck with citations; Bob GUI MCP cancer/insulin exact refusals with GET read-back + verbatim Bob MCP transport evidence), historical 429 quota blocker on `models/gemini-embedding-001` resolved after operator global-credential update, native bypass runs (pre-repair unapproved dual-edge failure vs post-fix 4-span clean refusal, 3-span red-flag/invalid), pure verifier boundary tests (constructed context), stored review-record immutability, 42-flow inventory audit, complete raw evidence appendix, and fresh all-5 export audit.
* [`exports/answer_care_questions.json`](exports/answer_care_questions.json): Fresh official MCP export of F5 (11 nodes, 10 edges post-T4 single-edge fix; live RAG route ACCEPTED 2026-10-05; global api_key references masked as `***REDACTED***` by the export tool — rebind to the existing `GOOGLE_API_KEY` global after import).

---

## 3. Five-Flow Pipeline Architecture & Implementation Status

| Flow ID | Flow Name | Purpose & Scope | Implementation Status |
|---|---|---|---|
| **F1** | `register_discharge_summary` | Ingests discharge summary text, extracts clinical episode parameters via LLM, and calculates a deterministic DEMO risk tier (LOW / MEDIUM / HIGH). **NOT a calibrated 30-day prediction model**. | **PASSED (Linux Host & Historical Windows)** — fixture 7/HIGH; isolated REST session |
| **F2** | `record_patient_checkin` | Ingests daily check-in text, detects 5 red-flag categories (immediate escalation), tracks non-flag symptom trajectory / adherence, calculates next check-in interval, and triggers plan review on tier changes. | **PASSED (Linux Host & Historical Windows)** — all 3 isolated cases; structural bypass proven |
| **F3** | `draft_followup_plan` | Generates a guarded Indonesian JSON draft with `satuseshat` (`careplan`, `servicerequest`, `task`) and mandatory human-approval status; fails closed on missing facts. | **PASSED (Linux Host & Historical Windows)** — HIGH draft and missing-facts cases |
| **F4** | `review_care_plan` | Human-in-the-loop review via `HumanInput`, approve-only SQLite persistence (`approved_plans`, `audit_log`), and routing (`Approve`, `Request Changes`, `Escalate`). | **PASSED (Linux Host & Historical Windows)** — actual v2 pause/resume; all 3 human actions; STOP 2 reached |
| **F5** | `answer_care_questions` | Grounded patient Q&A over APPROVED care plan and protocol chunks via Chroma vector retrieval with exact embedding model `models/gemini-embedding-001`, strict out-of-scope refusal, and red-flag bypass. | **PASSED (Linux Host, 2026-10-05)** — Flow [`4fff7e76-d710-4662-9b8e-087b5182a245`](exports/answer_care_questions.json) (11 nodes, 10 edges); native bypass runs verified; live RAG route proven: in-scope diet keycheck with citations 13:38 UTC and Bob GUI MCP out-of-scope cancer + out-of-plan insulin exact refusals 14:26 UTC, after operator global-credential update (agent never touched key values); synthetic fixture acceptance only (see [`reports/answer_care_questions-test.txt`](reports/answer_care_questions-test.txt) Section 10) |

> **Flow ID Notice (Historical Windows vs Current Linux Live Instance)**:
> - **F1 (`register_discharge_summary`)**: Live Linux UUID `dd096aae-e02c-4a36-b3c3-ec8c741b0e66` (historical Windows: `458d7c88-812b-4cb2-bc3f-ccccf809ed1a`; oldest historical absent: `e1267720-b42b-4bcf-aff5-1792a4871df1`).
> - **F2 (`record_patient_checkin`)**: Live Linux UUID `22698f53-9f32-4fc9-acc2-c53efdcd9f58` (historical Windows: `9b6cc0a6-7006-4f07-86e9-f47b61c9b462`).
> - **F3 (`draft_followup_plan`)**: Live Linux UUID `fb72ad45-b2ac-4979-a34c-c5e2f0725754` (historical Windows: `bd4488e6-e8fb-4107-b497-1b5178bc8371`).
> - **F4 (`review_care_plan`)**: Live Linux UUID `b9413d2e-cb01-43e0-b83b-d4b1bb90a3a3` (historical Windows: `83d39915-9176-4cbd-aac5-74dd00461bf4`).
> - **Server Inventory & Nodes**: Current Linux server inventory contains 42 flows (exactly 5 SynCare flows, 37 preserved unrelated user/starter flows). SynCare flows: F1 (4 nodes, 3 edges), F2 (7 nodes, 6 edges), F3 (6 nodes, 6 edges), F4 (12 nodes, 11 edges), F5 (11 nodes, 10 edges; flow ID `4fff7e76-d710-4662-9b8e-087b5182a245`). All five flows now have fresh official exports in `exports/` (2026-10-05: F5 re-exported after acceptance, SHA-256 fa0d8320bc477abcd...; F1–F4 re-exported unchanged with byte-derived manifest hashes in the export audit). All five flows passed synthetic fixture acceptance; F5 live RAG route ACCEPTED.
> - **Credential Binding**: Live Linux flows bind global credentials via `load_from_db: true` (target credential exists in the Linux Langflow global store; historical `GEMINI_API_KEY` is absent on Linux). Live flow projections reference the existing `GOOGLE_API_KEY` global (F1/F2/F3 generator key fields; F5 R1+V1 embedding/generator key fields); the official export tool masks these references as `***REDACTED***` in exported JSON — operators must REBIND the masked api_key fields to the existing `GOOGLE_API_KEY` global after any fresh import (all five masked key fields: F1, F2, F3 one each; F5 two). Agents never inspected or changed credential values; the operator updated the global credential value directly before the passing F5 runs.
> - **Review Database**: Authorized target Linux review database created at `/home/frxskie/langflow/syncare-review.sqlite3` (outside git repo; 1 approved plan, 3 audit records; historical Windows DB `C:/Users/Frxsk/langflow/syncare-review.sqlite3` retained with 2 approved, 4 audit as labeled historical evidence).
---

## 4. F1 (`register_discharge_summary`) — Passed Fixture Acceptance & Repair History

### 4.1 Expected Pipeline & Deterministic Scoring
1. **Input**: Synthetic Indonesian discharge summary text (`fixtures/discharge-chf-high.txt`: 62M, CHF NYHA III, LOS 6 days, emergency admission via IGD, DM2 + Hypertension + CKD stage 3 [3 comorbidities], 2 admissions in past 12m).
2. **LLM Extraction**: Exact type `ext:google:GoogleGenerativeAIComponent@official`, model `gemini-3.5-flash-lite` per operator decision **2026-10-03**, explicit `temperature: 0`. It is absent from the registry's static option list, but `model_name.combobox: true` permits this custom value; inspection confirmed it exactly and real execution succeeded.
   * **Episode schema**: Exactly `diagnosis` (string), `medications`, `instructions`, `followup_needs` (arrays of stated strings), `length_of_stay_days`, `comorbidity_count`, `admissions_prior_12m` (nonnegative integer or `null`), and `admission_type` (`emergency`, `elective`, or `unknown`). IGD/emergency maps to `emergency`; absent facts remain empty/null/unknown, never invented.
   * **Credential binding proven**: Live Linux flow references `api_key.value: "GOOGLE_API_KEY"` with `load_from_db: true`; successful Gemini execution on Linux establishes live runtime binding. Historical Windows execution succeeded separately with `"GEMINI_API_KEY"`. This is a variable NAME, not a credential value; no global variables were read or changed. MCP inspection/export redacts this reference, which was left untouched.
3. **Deterministic Scorer (`demo-heuristic-v1`)**:
   * Length of Stay (LOS) $\ge 5$ days: **+2 points**
   * Emergency admission (`masuk via IGD`): **+1 point**
   * Comorbidities $\ge 3$: **+2 points**
   * Prior hospitalizations in last 12 months $\ge 2$: **+2 points**
   * **Risk Tiers**:
     * **LOW**: 0–1 points
     * **MEDIUM**: 2–3 points
     * **HIGH**: $\ge 4$ points
   * **Expected Fixture Output**: $2 + 1 + 2 + 2 = 7$ points $\rightarrow$ **HIGH** tier (all 4 factors present).

### 4.2 Passed Validation & Exact Fixture Runs
* **Live Linux Host Run (2026-10-05)**: Flow `dd096aae-e02c-4a36-b3c3-ec8c741b0e66`. `validate_flow` passed with `{"valid": true, "component_count": 4, "errors": []}`. Live REST fixture run `POST /api/v1/run/dd096aae-e02c-4a36-b3c3-ec8c741b0e66` with top-level `session_id: "demo-linux-chf-high"` and tweaks for `ChatInput-sapT9` and `ChatOutput-XJH2z` yielded `json.loads`-valid ChatOutput text with score **7**, tier **HIGH**, all four factors met, full eight-field episode, and label `demo-heuristic-v1`. Response, message, and asynchronous trace `629ddc40-9d62-4cd6-a469-ef9aceb5c1ef` all confirmed session `demo-linux-chf-high` with status `ok`.
* **Historical Windows Run (2026-10-03)**: Flow `458d7c88-812b-4cb2-bc3f-ccccf809ed1a`. `validate_flow` returned `{"valid": true, "component_count": 4, "errors": []}`. REST fixture run with top-level `session_id: "demo-chf-high"` and tweaks for historical nodes `ChatInput-wQRXg` and `ChatOutput-dkoVg` isolated all three observed IDs to `demo-chf-high` (trace `42303d8a-301f-429b-8ba1-52a0d4f0332e`).
### 4.2.1 Resolved Model Availability Blocker (History)
Before the operator's explicit model switch, one `validate_flow` call returned:

```json
{
  "valid": false,
  "component_count": 1,
  "errors": [
    {
      "component_id": "flow",
      "error": "Build error"
    }
  ]
}
```

Read-only REST trace discovery isolated that historical current-instance failure. Trace `9441f3c0-6502-41cd-8b2e-71349aeecac7`, timestamp `2026-10-03T12:18:04.560578`, records:

```text
Error calling model 'gemini-2.5-flash' (NOT_FOUND): 404 NOT_FOUND. {'error': {'code': 404, 'message': 'This model models/gemini-2.5-flash is no longer available to new users. Please update your code to use models/gemini-3.8-flash for the latest features and improvements. We recommend you to use the Interactions API (https://ai.google.dev/gemini-api/docs/get-started).', 'status': 'NOT_FOUND'}}
```

At that blocked checkpoint only ChatInput had a fresh build; downstream successful records were stale, and no additional validation/run was attempted. The operator subsequently authorized: **"Let's try gemini-3.5-flash-lite. This model is relatively new and has a large window for free usage."** This changed only the F1/F2 model requirement, retaining temperature 0. The new model passed actual validation and fixture execution; the old 404 is retained solely as history.

### 4.3 Implemented Structure & Verification
* Current runtime/native export identifies `lfx` **1.12.4**, not the prior instance's 1.12.2. The pre-existing current scorer already used `lfx.custom`, `lfx.io`, and `lfx.schema.message`; these supported imports were retained.
* The scorer strips JSON markdown fences, requires exactly all eight episode keys, validates value types, permits null/unknown facts, and retains the complete `episode`. It emits `score`, `tier`, all four `risk_factors` (value/threshold/points/met), `rationale`, `label: "demo-heuristic-v1"`, `missing_fields`, and synthetic/not-clinically-validated disclaimers via `Message(text=json.dumps(...))`.
* `get_component_info` confirms registered `output.method: "score_risk"` and `input_value: ""`; no `"Hello, World!"` parameter remains. Existing inputs, outputs, and edge handles were preserved. Installed MCP source shows code configuration alone does **not** re-derive the template; full custom-code template derivation is available through `/api/v1/custom_component`. No registration mismatch existed on this instance.
* ChatOutput's `clean_data` is explicitly false. Native `safe_convert` returns Message text directly; Data serialization can introduce markdown fences. The final Message-based scorer output is now exercised end-to-end and parseable without stripping output fences.
* Authorized throwaway local checks: exact scorer code compiled and imported/instantiated with the installed server environment. Hand-written fixture-shaped extraction yielded **7/HIGH**, all four factors met; all-unknown extraction yielded **0/LOW**, with eight missing fields. Both fenced input cases produced JSON text parseable with `json.loads`. These are **local scorer unit smokes, not Gemini extraction or flow runs**. Temporary scripts/bytecode were deleted.
* `notify_done` acknowledged the passed summary after the earlier blocked checkpoint. The current export is the unmodified redacted `export_flow` tool output; the transcript retains both checkpoints and every actual mutation/run result.

### 4.4 Historical Old-Instance Failure (Unprovable Here)
On original ID `e1267720-b42b-4bcf-aff5-1792a4871df1`, two validations returned the same opaque `"Build error"` and only ChatInput recorded a build. The old export stored `langflow.*` scorer imports while native sources used `lfx` 1.12.2, stale method `build_output`, `"Hello, World!"`, empty API reference, and a deviating extraction schema. These are historical observations; the exact old failure cause remains unproven. The current model-404 trace does **not** establish the root cause on that absent instance.

---

## 5. Flow Specifications & Operator Stop Phrases

### 5.1 Operating Discipline & Multi-Agent Stop Phrases
* **Per-Flow Lifecycle**: discover $\rightarrow$ one-shot create $\rightarrow$ validate $\rightarrow$ real run $\rightarrow$ `notify_done` $\rightarrow$ redacted export + actual transcript $\rightarrow$ atomic commit.
* **Error Discipline**: Two identical consecutive errors $\rightarrow$ **STOP immediately**.
* **Safety Mandates**: Preserve other user flows on the server; never store or commit literal API secrets.
* **Authorized Scope & Status**: F1–F4 passed on both the live Linux host (`http://127.0.0.1:7860`) and historical Windows machine on operator-authorized `gemini-3.5-flash-lite` through STOP 2. Linux review database persisted at `/home/frxskie/langflow/syncare-review.sqlite3` with exactly 1 approved plan and 3 audit logs (stored records unchanged across all F5 acceptance runs). On 2026-10-05, operator issued STOP 2 resume phrase `checkpoint-2-human-gate selesai, lanjut` authorizing F5; generator model confirmed `gemini-3.5-flash-lite` at temperature 0.2; F5 flow created (`4fff7e76-d710-4662-9b8e-087b5182a245`, 11 nodes, 10 edges post-T4 fix). F5 is now **ACCEPTED** (2026-10-05): after the operator updated the global Gemini credential (agents never inspected or changed credential values), the unchanged flow code/model settings passed the live RAG route — diet keycheck 13:38 UTC, cancer + insulin exact refusals 14:26 UTC. Five separate flows passed synthetic fixture acceptance; no integrated end-to-end 5-flow pipeline execution is claimed.

**Exact Stop & Resume Phrases (Do NOT invent dialogue)**:
1. **STOP1** (after completing F1 and F2): Completed and resumed on 2026-10-04 upon operator command:
   ```text
   checkpoint-1-mvp-intake selesai, lanjut
   ```
2. **STOP2** (after completing F3 and F4): Resume upon operator command:
   ```text
   checkpoint-2-human-gate selesai, lanjut
   ```
   *(Received 2026-10-05: F5 authorized; F5 subsequently ACCEPTED the same day after the operator global credential update resolved the embedding 429 blocker; STOP 3 COMPLETE — halt and await further operator instructions; no extra flows)*
3. **STOP3** (after completing F5, inventory verification, and exporting all 5 flows): **Halt and await further operator instructions; create nothing new**.

---
### 5.2 Handoff Next Actions (Immediate Sequence for Incoming Agent)
1. **F1 Accepted (Linux Host & Historical Windows)**: Verified passed on Linux host (`POST /api/v1/run/dd096aae-e02c-4a36-b3c3-ec8c741b0e66` with tweaks `ChatInput-sapT9`, `ChatOutput-XJH2z`, session `demo-linux-chf-high`), yielding 7/HIGH with full 8-field episode under live `GOOGLE_API_KEY` credential binding. Preserved model `gemini-3.5-flash-lite`, temperature 0, and deterministic scoring contract.
2. **F2 Accepted (Linux Host & Historical Windows)**: Verified passed on Linux host (`POST /api/v1/run/22698f53-9f32-4fc9-acc2-c53efdcd9f58` with tweaks `ChatInput-WtGHx`, `ChatOutput-gDqyg`, `ChatOutput-mgriH`, and scorer `CustomComponent-uFKzq` `baseline_score`/`patient_id`), proving baseline reduction (Case 1: 6/HIGH, `continue_monitoring`), tier escalation (Case 2: 3->6/HIGH, `trigger_plan_review`), and structural red-flag bypass (Case 3: `escalate_immediately`, formatter executed and scorer completely excluded with unchanged build timestamp).
3. **Sessions & Audit**: Top-level REST `session_id` confirmed isolating response, message, and trace across Linux runs (`demo-linux-chf-high`, `demo-linux-checkin-medium`, `demo-linux-checkin-redflag`). Component tweaks alone were demonstrated ineffective historically on Windows; Linux REST routing enforces session isolation. Trace writes are asynchronous: match by actual session/run timestamp.
4. **F1–F5 Verified Linux PASSED; STOP 3 COMPLETE**: All five flows are verified passed on the Linux host with authentic executions and Linux SQLite persistence. F1–F4 runs are REST session-isolated; F5 runs evidence is correlated by actual query, timestamp, and trace IDs under the shared MCP session (flow UUID; no false F5 session isolation). F5 flow (`4fff7e76-d710-4662-9b8e-087b5182a245`, 11 nodes, 10 edges post-T4 fix) was ACCEPTED on 2026-10-05: native 4 bypass cases passed earlier, and after the operator updated the global Gemini credential (agents never inspected or changed credential values), the live RAG route passed the in-scope diet keycheck (13:38 UTC) plus out-of-scope cancer and out-of-plan insulin exact refusals (14:26 UTC, Bob GUI MCP transport, monitor-trace verified). Fresh official all-5 exports and `notify_done` are complete; STOP 3 COMPLETE — halt and await further operator instructions; no extra flows. Five separate flows passed synthetic fixture acceptance — no integrated end-to-end pipeline run is claimed.

---


### 5.3 Flow F2: `record_patient_checkin` — Contract & Actual Acceptance
* **Input Extraction**: Gemini `gemini-3.5-flash-lite`, `temperature: 0.0`, per operator decision **2026-10-03**.
* **Output Schema**: JSON object with:
  * `symptoms`: array of symptom strings
  * `medication_adherence`: `"regular"` | `"irregular"` | `"unknown"`
  * `red_flag`: boolean
  * `quote_of_red_flag`: string (extracted exact quote if red flag present)
* **Symptom Granularity**: Prompt must distinguish mild complaints (e.g., `'sedikit sesak'`) from severe red flags (e.g., `'sesak napas berat'`).
* **Exact Five Red-Flag Categories**:
  1. Chest pain (`nyeri dada`)
  2. Severe shortness of breath (`sesak napas berat`)
  3. Fainting / loss of consciousness (`pingsan / hilang kesadaran`)
  4. Heavy bleeding (`perdarahan hebat`)
  5. Suicidal statements (`pernyataan ingin mengakhiri hidup`)
* **Red-Flag Branching**: If `red_flag == true`, flow **MUST skip the risk scorer** and output:
  ```json
  {"action": "escalate_immediately", "reason": "<quote_of_red_flag>"}
  ```
  *(Note: Must NOT output autonomous emergency contact instructions).*
* **Non-Flag Numeric Scoring Rule**:
  * Base score = baseline discharge score.
  * If new `symptom_count >= 2`: **+2 points**
  * If `medication_adherence == "irregular"`: **+1 point**
  * If `symptoms` array is empty: **-1 point** (floor at 0).
  * **Risk Tiers**: HIGH ($\ge 4$), MEDIUM (2–3), LOW (0–1).
  * **Check-in Interval**: `next_checkin_days` $\rightarrow$ HIGH = 1 day, MEDIUM = 2 days, LOW = 7 days (configurable DEMO parameters, not clinical guidance).
  * **Tier Change Action**: If tier changes from baseline, trigger action: `"trigger_plan_review"`.
* **Batch Acceptance Test Suite (All 3 required)**:
  1. `'Merasa lebih baik, obat teratur'` $\rightarrow$ Baseline HIGH remains HIGH or shows visible reduction.
  2. `'Hari ini masih sesak napas, kaki kanan bengkak, minum obat tidak teratur.'` $\rightarrow$ MEDIUM $\rightarrow$ HIGH transition with `"trigger_plan_review"`.
  3. `'dada sesak berat sekali'` $\rightarrow$ Escalation output WITHOUT passing through risk scorer.


#### 5.3.1 Implemented Graph & Registration
The seven-node/six-edge graph is `ChatInput → Gemini → ConditionalRouter`, then:
* `false_result → SynCare Deterministic Check-in Scorer → normal ChatOutput`
* `true_result → SynCare Red-flag Escalation Formatter → red-flag ChatOutput`

The native router matches JSON boolean `red_flag: true` with regex `(?s).*"red_flag"\s*:\s*true\b.*` and passes the original extraction JSON. Native `lfx` 1.12.4 routing calls both `stop()` and `exclude_branch_conditionally()` for the inactive branch. Each branch has its own terminal ChatOutput; only the active output is returned. The scorer never substitutes for red-flag routing and rejects a flagged payload if it is incorrectly reached.

Custom component source was registered via `/api/v1/custom_component`, then persisted into the two F2 nodes so inputs/outputs were genuinely derived from code. Output handles were regenerated through MCP disconnect/connect calls. Scorer method is `score_checkin`; formatter method is `format_escalation`. Both emit `Message(text=json.dumps(...))`. Model/temperature inspection confirmed `gemini-3.5-flash-lite`/0. Flow metadata references `GOOGLE_API_KEY` on live Linux (historical Windows: `GEMINI_API_KEY`) with `load_from_db: true`; actual Gemini-backed runs succeeded without reading or editing globals.

#### 5.3.2 Actual Tweaks & Isolated Sessions

**Live Linux Host Tweak Map & Runs**:
Use the approved REST `POST /api/v1/run/22698f53-9f32-4fc9-acc2-c53efdcd9f58` with top-level `session_id: "demo-linux-<patient_id>"`. Component tweak keys are:
* Input `ChatInput-WtGHx`: `session_id`
* Normal output `ChatOutput-gDqyg` and red output `ChatOutput-mgriH`: `session_id`
* Scorer `CustomComponent-uFKzq`: integer `baseline_score`, string `patient_id`
* Configurable advanced demo intervals on the scorer: `high_interval_days: 1`, `medium_interval_days: 2`, `low_interval_days: 7` (not clinical guidance).

Messages are not persisted (`should_store_message: false` on input and both outputs). Baselines are supplied separately from check-in text. Baseline **7** is the actual accepted F1 score; baseline **3** is the explicitly supplied synthetic MEDIUM case, not a newly calculated clinical baseline.

| Patient / observed message and trace session | Baseline | Actual JSON result (Live Linux Host) |
|---|---:|---|
| `chf-high` / `demo-linux-chf-high` | 7 | No symptoms, regular adherence: **6/HIGH**, visible -1, `continue_monitoring`, interval 1 (trace `7ce6dce6-1ba5-46ee-8617-c2689b7a17e4`) |
| `chf-medium` / `demo-linux-checkin-medium` | 3 | Two symptoms, irregular adherence, **not red-flagged**: **6/HIGH**, MEDIUM→HIGH, `trigger_plan_review`, interval 1 (trace `e45362e9-6857-4f81-bd52-dd1081a0c51f`) |
| `chf-redflag` / `demo-linux-checkin-redflag` | 7 | Exactly `{"action": "escalate_immediately", "reason": "dada sesak berat sekali"}`; no scorer execution (trace `563dafc2-3ebb-4720-a2f0-674a083b6d9a`) |

**Historical Windows Host Tweak Map & Runs (Labeled Historical)**:
On historical Windows flow `9b6cc0a6-7006-4f07-86e9-f47b61c9b462`, historical tweak nodes were `ChatInput-IW816`, `ChatOutput-pTVwm`, `ChatOutput-BdCAF`, and `CustomComponent-0WfNa`, run with sessions `demo-chf-high`, `demo-chf-medium`, `demo-chf-redflag`.

Both Linux and historical validations passed with `{"valid": true, "component_count": 5, "errors": []}`: five active vertices out of seven total is expected for one selected branch. All three final texts passed `json.loads`. The non-red payload includes baseline/current tiers, each adjustment's condition/applied/points, score, tier change, action, interval, complete extraction, `demo-heuristic-v1`, and disclaimers.

#### 5.3.3 Structural Bypass Evidence
* **Live Linux Verification (2026-10-05)**: Red-flag trace `563dafc2-3ebb-4720-a2f0-674a083b6d9a` (`demo-linux-checkin-redflag`) contains exactly 5 spans: `Chat Input`, `Google Generative AI`, `If-Else`, `SynCare Red-flag Escalation Formatter`, and `Chat Output`, with **no scorer span** (`CustomComponent-uFKzq`), including children. The scorer's retained build timestamp is unchanged from the prior medium run (`2026-10-05T07:57:59.381105Z`), before red-flag trace start `2026-10-05T07:58:05.850884Z`. This proves complete structural bypass on Linux.
* **Historical Windows Evidence (2026-10-03)**: Red-flag trace `70a44e8b-cc76-45dc-98cc-96c85c4ab091` (`demo-chf-redflag`) similarly proved bypass on the historical Windows instance.
---

### 5.4 Flow F3: `draft_followup_plan` Specifications
* **LLM Engine**: Gemini `gemini-3.5-flash-lite`, `temperature: 0.2`.
* **Model Selection Note**: `gemini-2.5-flash` returns provider 404 on this account. F3 uses `gemini-3.5-flash-lite` at temperature 0.2, extending the operator's F1/F2 model choice while keeping the F3 spec temperature 0.2 unchanged.
* **Implementation Status**: **PASSED (Linux Host & Historical Windows)**. Live Linux flow `fb72ad45-b2ac-4979-a34c-c5e2f0725754` passed live REST acceptance runs (HIGH draft with care intensity and SATUSEHAT resources; fail-closed missing facts with structural Gemini bypass) and live MCP validation. Facts/source scheduling synthetic parameters (`2026-10-11`, `HOTLINE-DEMO-SINTETIS`) remain unchanged. Historical Windows flow `bd4488e6-e8fb-4107-b497-1b5178bc8371` passed on 2026-10-04.
* **Care Intensity Parameters (Demo Guidelines)**:
  * **HIGH**: Nurse phone call within 24 hours + daily check-ins.
  * **MEDIUM**: Check-in every 2 days + outpatient clinic visit in 7 days.
  * **LOW**: Weekly check-in + outpatient clinic visit in 30 days.
* **Output Format**: Strictly valid JSON with:
  * `plan_text`: Care plan narrative in Bahasa Indonesia.
  * `satuseshat`: Object containing FHIR-shaped resources:
    * `careplan`: `{"title": "...", "intent": "...", "activity": []}`
    * `servicerequest`: `{"patientInstruction": "follow-up instruction including supplied date and hotline", "priority": "..."}` *(Note: hotline is a routine follow-up contact, NOT emergency instructions; do NOT invent phone numbers or dates—use supplied actual data)*.
    * `task`: array of task objects: `[{"code": "...", "status": "requested", "priority": "..."}]`
* **Guardrails**: Guard against absent episode facts; do **NOT** invent diagnoses or medications. Require supplied actual data.
* **Status**: All generated plans marked **DRAFT pending human approval**.

#### F3 Implemented Guardrails & Actual Acceptance
Input is one JSON object with `f1_scored_output` (the complete actual F1 output), `patient_id`, `followup_date`, and `hotline`. The HIGH fixture preserves the actual accepted F1 ChatOutput JSON bytes from its transcript line 3669, then supplies synthetic date `2026-10-11` and explicitly non-dialable routine contact `HOTLINE-DEMO-SINTETIS`. The missing fixture keeps the same patient but empties medications and omits both scheduling facts.

Graph: `ChatInput → deterministic Plan Fact Gate → Gemini → deterministic Draft Guard → ChatOutput`, with a separate `Fact Gate.error → error ChatOutput`. Before generation, the gate validates the eight-field episode, score/tier/label agreement and scheduling facts, and injects HIGH/MEDIUM/LOW intensity deterministically. Missing facts emit explicit error JSON and structurally bypass Gemini rather than inventing a partial plan.

The real Gemini call at temperature 0.2 selects one of two approved Indonesian narratives and fills the exact injected resource structure. The post-generation guard enforces exact resource equality and a closed factual narrative vocabulary: arbitrary invented medication/diagnosis strings fail even if a medical-keyword blacklist would miss them. Date/phone-pattern checks provide defense in depth. This deliberately bounds phrasing freedom; reviewer notes can choose a clearer approved narrative but cannot change clinical facts. Accepted output includes `status: "DRAFT pending human approval"`, `label`, disclaimer, `missing_facts: []`, deterministic `care_intensity`, `patient_id`, `tier`, and preserved `source_input` for reviewed redrafting. Resources are FHIR-shaped demo objects, **not** a validated FHIR bundle or SATUSEHAT submission.

Validation: `{"valid": true, "component_count": 5, "errors": []}` (five active vertices of six total).
* HIGH test (`demo-chf-high`, trace `414bf0f6-65fb-471a-af19-9c7b325ebb15`): valid JSON, DRAFT status, nurse-call-within-24h and daily check-ins, all required `satuseshat` keys, exact supplied date/contact in `patientInstruction`.
* Missing test (`demo-chf-missing`, trace `79b83a91-ae4b-4eff-b8d1-70f325f4525b`): explicit `missing_or_invalid_facts` JSON with `missing_facts: ["medications", "followup_date", "hotline"]`; no invented facts and no Gemini span in the current trace.

Credential binding references `api_key.value: "GOOGLE_API_KEY"` on live Linux (`"GEMINI_API_KEY"` on historical Windows) with `load_from_db: true`, never a literal key. Full verbatim evidence is in the F3 transcript.


---

### 5.5 Flow F4: `review_care_plan` Specifications
* **Implementation Status**: **PASSED (Linux Host & Historical Windows)**, live flow `b9413d2e-cb01-43e0-b83b-d4b1bb90a3a3` (historical Windows flow: `83d39915-9176-4cbd-aac5-74dd00461bf4`); STOP 2 complete on both machines. Authorized via STOP 1 resume phrase `checkpoint-1-mvp-intake selesai, lanjut` on 2026-10-04.
* **Human-in-the-Loop Component**: Native `HumanInput`, with actual observed `human_input_required` pause and resumed human decisions.
* **Configured Actions**: `Approve`, `Request Changes`, `Escalate`; native `fallback` routes to its own Escalate handler. Timeout is `5 Minutes` = **300s**, confirmed in each pause event. Timeout rerouting is evaluated on a late response; automatic escalation exactly at 300s is not claimed. Expiry/fallback execution was not acceptance-tested.
* **Action Logic**:
  * `Approve`: Writes approved plan record to SQLite table `approved_plans(plan_json, approved_at, reviewer_note)` + outputs confirmation message in Bahasa Indonesia.
  * `Request Changes`: Audits the actual critique, passes preserved factual source input plus `reviewer_note` to native `RunFlow` invoking the existing F3, and returns a materially revised **DRAFT**. This is one real redraft, not automatic approval or a cyclic graph edge; resubmit the returned draft to F4 for a new human decision.
  * `Escalate`: Emits escalation payload: `{"action": "escalate_to_clinician", "plan_json": {...}}`.
* **Audit Trail**: All three actions append rows to SQLite table `audit_log(action, actor, timestamp, payload)`.
* **Integrity Invariant**: **NOTHING** is written to `approved_plans` prior to explicit human approval.
* **Acceptance Tests**: All three actions completed through the v2 background API, with actual pause/resume, isolated sessions, per-run SQLite counts, and material nonclinical critique response.

#### F4 Implemented Gate, Persistence & Actual Acceptance
Input is the complete actual guarded F3 HIGH ChatOutput JSON. The intake component validates its DRAFT status, label, preserved source input and resources, and initializes empty tables only. Each action handler reads the real injected `graph.human_input_decisions[<HumanInput_id>:<run_id>]` dictionary (live Linux: `HumanInput-IvOz5`; historical Windows: `HumanInput-Gy29k`), including `actor` and `reviewer_note`. Missing/mismatched decisions refuse all inserts. Approve writes its approved plan and audit row in one SQLite transaction; all other actions write audit rows only. Database path: authorized Linux review DB is **`/home/frxskie/langflow/syncare-review.sqlite3`** (historical Windows path: `C:/Users/Frxsk/langflow/syncare-review.sqlite3`), outside this repository. Schema is `approved_plans(plan_json TEXT, approved_at TEXT, reviewer_note TEXT)` and `audit_log(action TEXT, actor TEXT, timestamp TEXT, payload TEXT)`.

**Live Linux Host Acceptance (2026-10-05)**:
In-place repairs re-bound all 4 action handlers to live gate `HumanInput-IvOz5`, configured review DB to `/home/frxskie/langflow/syncare-review.sqlite3`, and derived RunFlow native template linked to live F3 `fb72ad45-b2ac-4979-a34c-c5e2f0725754`. Chronology of the one failed job (exact id `909d276d-7d8d-4317-9a81-0d8ebdd3afb6`, FAILED ~14s, no DB rows existed): it occurred AFTER the RunFlow reference cutover, when the `/api/v1/custom_component/update` response's derived `code` field carried `input_types: null` (lfx `Input.model_dump` serialization), which `lfx` `parse_data` (`lfx/graph/vertex/base.py:280`) cannot consume — `TypeError: 'NoneType' object is not iterable` before any vertex ran. The first-run SSE capture recorded 46 replayed error events with a single distinct error text (event replay during bounded polling, not 46 distinct failures; native event ids were not captured by the first-pass reader). Resolved by removing that single null key from the node descriptor only (RunFlow source value untouched); NOT an error suppression and NOT a defect of the original stale template. All three subsequent actions completed via v2 background API with genuine `human_input_required` pauses (timeout 300s):

| Final case | Job / trace ID | Session | Approved before → paused → after | Audit before → paused → after |
|---|---|---|---|---|
| Approve | `25db3825-5638-4b3c-a19a-bfda15623de3` | `demo-chf-approve-linux` | DB absent → 0 → 1 | DB absent → 0 → 1 |
| Request Changes | `f3457189-8f99-42f7-a0ff-24cff7beacb4` | `demo-chf-request-changes` | `1 → 1 → 1` | `1 → 1 → 2` |
| Escalate | `777532c2-65ab-4f8a-a9e8-5c231a2f539c` | `demo-chf-escalate` | `1 → 1 → 1` | `2 → 2 → 3` |

Final counts on Linux: exactly 1 approved plan, 3 audit records. Approve inserted approved plan and audit log with identical timestamps (`2026-10-05T08:21:37.002846+00:00`). Note: before the Approve run the review DB file did not exist; the (0,0) "before" state is inferred from DB-absence, not a queried zero. Request Changes executed nested F3 through native RunFlow (child trace `84c0db01-65f1-4fe3-a196-9654a312a031`, 1582 tokens, status `ok`, child session matching parent `demo-chf-request-changes`), materially changing the opening to **“Ringkasan rencana pemantauan.”** while the remaining `plan_text` is byte-identical (substring equality after the opening) and all clinical facts/resources are exactly equal (semantic equality of parsed values), status DRAFT. Escalate emitted exact `{"action": "escalate_to_clinician", "plan_json": ...}`. Zero inserts into `approved_plans` occurred before or outside Approve.

**Approval-Status Nuance**: `approved_plans` table membership + `approved_at` + `reviewer_note` + audit row defines the workflow approval state. The stored `plan_json` retains status 'DRAFT pending human approval' (original reviewed artifact preserved verbatim by design; approve handler does not rewrite the plan's status field). Any future approved-only retrieval (e.g. F5) must query on `approved_plans` table membership, NOT `plan_json.status`. Synthetic workflow approval is NOT clinical authorization.

**Historical Windows Host Acceptance (2026-10-04, Labeled Historical)**:
On historical Windows flow `83d39915-9176-4cbd-aac5-74dd00461bf4`, gate `HumanInput-Gy29k` and database `C:/Users/Frxsk/langflow/syncare-review.sqlite3`, counts progressed from an initial authorized row retained from an earlier failed run: Approve `ba5111b9...` (1,1)->(2,2); Escalate `a7de048a...` (2,2)->(2,3); Request Changes `206617e3...` (2,3)->(2,4) with nested revision trace `3fce0dc6-8901-41a4-bb45-d05f6174f0f7`.

**Execution transport**: `POST /api/v2/workflows` with `mode:"background"` and top-level `session_id`; read `/api/v2/workflows/<job_id>/events`, then resume with `POST /api/v2/workflows/<job_id>/resume` and `{"request_id":<actual pause request>,"decision":{"action_id":"approve|request_changes|escalate","actor":<synthetic reviewer>,"reviewer_note":<actual critique>}}`. Native RunFlow's session was explicitly configured to the revision session; future callers should supply `tweaks["RunFlow-rGQR7"]["session_id"]` (Linux) or historical `RunFlow-m412w` for their own child redraft session.

**Validation limitation**: MCP `validate_flow` uses v1 direct build, which does not arm the durable background pause seam and fails on the placeholder return; v1 `run_flow` also rejects HITL outright. The completed v2 background builds, real pauses and completed resumes are the acceptance evidence. Server `notify_done` emitted status warning 'Event could not be delivered; UI will update after timeout' which is non-fatal.


---

### 5.6 Flow F5: `answer_care_questions` Specifications
* **Vector Store & Embeddings**: Astra DB vector backend (primary) with local Chroma fallback — same enforced EXACT embedding model `models/gemini-embedding-001` (no alternate or fallback embedding models); backend selectable per run via the component's `vector_backend` field, with fail-closed automatic fallback to Chroma when Astra credentials (`ASTRA_DB_API_ENDPOINT` / `ASTRA_DB_APPLICATION_TOKEN` global variables) are unresolvable (live Astra acceptance verified 2026-10-06).
* **Retrieval Scope**: Top-4 retrieved chunks scoped strictly to the patient's APPROVED care plan and protocol documents.
* **Generator**: Operator authorized `gemini-3.5-flash-lite` at `temperature: 0.2` on 2026-10-05 (extending the F1/F2 decision; `gemini-2.5-flash` returns provider 404 on this account). Generates answers in Bahasa Indonesia with citations to source chunks.
* **Implementation Status**: **PASSED / ACCEPTED (Linux Host, 2026-10-05)**. Flow `4fff7e76-d710-4662-9b8e-087b5182a245` (11 nodes, 10 edges post-T4 single-edge fix). Authorized via STOP 2 resume phrase `'checkpoint-2-human-gate selesai, lanjut'` on 2026-10-05 with generator `gemini-3.5-flash-lite` (temp 0.2). Native bypass cases passed on live server: initial runs verified red-flag escalation, malformed query JSON error, and unapproved-redflag escalation; initial unapproved run produced refusal but revealed dual-edge leakage triggering Verifier, which was resolved by T4 single-edge fix, with post-fix re-run (`demo-f5-unapproved-after`) proving exactly one active refusal output and zero Verifier spans. The 429 RESOURCE_EXHAUSTED blocker on `models/gemini-embedding-001` was resolved after the OPERATOR updated the global Gemini credential (agents never inspected or changed credential values; no flow/config change); the unchanged flow then passed live RAG acceptance: in-scope diet keycheck with citations (13:38 UTC, trace `7a7212c6-bc49-4199-9bd3-7007860145e5`), out-of-scope cancer refusal (14:26:24 UTC, trace `20bc73e3-d19f-41a6-a66c-a67bf5716f80`), and out-of-plan insulin refusal (14:26:57 UTC, trace `9d5b871a-80b4-4eb6-9efe-82c5018cdcb1`) — all full RedFlagGuard→Retriever→Verifier paths, single active ChatOutput, diet cited answer; cancer + insulin exact refusals; all 1 output / error=false (report Section 10).
* **Extractive Whole-Sentence RAG Design**: Prohibits free clinical generation; the model is constrained to selecting verbatim whole units from retrieved approved plan chunks and unapproved educational protocol drafts (`protocols/hf-aftercare.txt`, `protocols/medication-safety.txt`).
* **Educational Protocols Disclaimer**: Documents in `protocols/` are generic educational drafts, NOT clinically reviewed or approved by medical staff.
* **Synthetic Approval Disclaimer**: Native F4 review approval represents synthetic workflow approval, NOT medical diagnosis or clinical authorization.
* **Out-of-Scope Refusal Guard**: If query falls outside retrieved context, output EXACT refusal string:
  ```text
  Maaf, pertanyaan itu di luar cakupan rencana Anda. Silakan tanyakan ke perawat Anda atau hubungi hotline.
  ```
* **Red-Flag Bypass**: If query expresses red-flag symptoms, bypass LLM generation and immediately output escalation action.
* **Review DB Integrity**: Review database `/home/frxskie/langflow/syncare-review.sqlite3` accessed strictly read-only (URI mode=ro) during all F5 executions; stored review records unchanged (approved_plans count 1, audit_log count 3, plan SHA256 and all three audit payload SHA256s unchanged; physical SQLite file byte identity not tested).
* **Live RAG Proof**: PROVEN 2026-10-05 — in-scope diet/lifestyle answering with citations (13:38 UTC keycheck), out-of-scope cancer refusal (14:26:24 UTC), and out-of-plan insulin refusal (14:26:57 UTC); full trace/read-back evidence in [`reports/answer_care_questions-test.txt`](reports/answer_care_questions-test.txt) Sections 10-11.
---

## 6. Manual Setup, Operator URL & Local Verification

### 6.1 Langflow Server URL & Flow Import
* **Server URL**: Configured connection is `http://127.0.0.1:7860`. Live Linux host environment: `/home/frxskie/langflow`, Langflow 1.12.2, lfx 1.12.2 / lfx-google 0.1.2, bridge lfx 1.12.4. (Historical Windows environment: Langflow 1.12.2, lfx 1.12.4). Existing credentials are used privately without printing or writing credential values.
* **Import Protection**: Check list_flows first. Import via UI only if named flow is absent; otherwise inspect/update existing flow; importer duplicate behavior is unverified. After importing any of the five exported flows, REBIND the masked `api_key` fields (value `***REDACTED***`, `load_from_db: true`) to the existing `GOOGLE_API_KEY` global variable — F1, F2, and F3 have one generator key field each; F5 has two (Retriever embedding key + Verifier generator key); F4 has none. Never embed a literal secret.

### 6.2 Local Artifact Smoke Test (No Workflow Execution)
To verify the structural integrity of the exported flow JSON:
```bash
python3 -c "import json; d=json.load(open('exports/register_discharge_summary.json')); print('Flow Name:', d.get('name'), '| Nodes:', len(d['data']['nodes']), '| Edges:', len(d['data']['edges']))"
```
*(Expected output: `Flow Name: register_discharge_summary | Nodes: 4 | Edges: 3`)*

> **Note**: This is a local file structure smoke test only, **NOT a workflow execution test**.

### 6.3 MCP `run_flow` Schema Limitation
The MCP schema has no top-level `session_id`. Historically on Windows, actual `run_flow` with ChatInput/ChatOutput `session_id: "demo-chf-high"` tweaks returned the **flow UUID** in its response, output message, and trace: the API's effective graph session overrides component tweaks. The operator approved direct REST `POST /api/v1/run/{flow_id}` with top-level `session_id` and the same tweaks. For the live Linux instance, the same REST transport is retained: runtime session isolation was fully verified and documented across flows F1–F4 (F1 `demo-linux-chf-high`, F2 `demo-linux-chf-high`/`medium`/`redflag`, F3 `demo-chf-high`/`missing`, and F4 `demo-chf-approve-linux`/`request-changes`/`escalate`), where REST response, message data, and trace session IDs all match the supplied top-level session ID. For F5, unique sessions were exercised in initial bypass runs, with targeted post-fix verification sessions pending handoff. Existing configured credentials are used privately; transcript headers are redacted. Message storage is disabled on intake and output nodes.

---

## 7. Deliverables & Screenshot Capture Plan

### Current Passed F1/F2 — Operator Capture Suggestions
1. F1 complete graph, Gemini `gemini-3.5-flash-lite`/temperature 0, and fixture output **7/HIGH** with all four factors and full episode.
2. F1 REST-run trace/message showing `demo-chf-high` (MCP tweaks alone used the flow UUID; do not present that run as isolated).
3. F2 complete graph showing separate normal-scoring and red-flag formatter branches.
4. F2 scorer fields/tweaks: `baseline_score`, `patient_id`, and configurable demo interval inputs.
5. F2 high and medium outputs: **6/HIGH** with -1, then **6/HIGH** with `trigger_plan_review` and `red_flag: false`.
6. F2 severe case: exact action/reason JSON, `demo-chf-redflag` trace showing formatter **without scorer**, and unchanged prior scorer build timestamp.

### Subsequent Flows (F3–F5) — Screenshot Capture Plan
Historical Windows and live Linux host runs documented passed executions for F1–F4 through STOP 2. F5 flow `4fff7e76-d710-4662-9b8e-087b5182a245` (11 nodes, 10 edges) was ACCEPTED on 2026-10-05 with live RAG proof (diet keycheck 13:38 UTC; cancer + insulin exact refusals 14:26 UTC via the Bob GUI test surface — validate_flow, two run_flow executions, notify_done status ok, and fresh export_flow all observed as actual MCP tool calls, with verbatim transport logs and monitor-trace read-backs preserved in the F5 report). Screenshot observations from the Bob GUI session were transient captures of that live test surface and are not claimed as permanent PNG deliverables; the primary evidence is the raw tool/trace record in [`reports/answer_care_questions-test.txt`](reports/answer_care_questions-test.txt). STOP 3 COMPLETE — halt and await further operator instructions; no extra flows.
