# SynCare — Risk-Adaptive Post-Discharge Monitoring Workflow (Hacktiv8 x IBM 2026)

## 1. Executive Summary & System Scope

**SynCare** is a risk-adaptive post-discharge patient monitoring prototype tailored for the Indonesian healthcare context, developed for the **Hacktiv8 x IBM 2026** initiative. The planned workflow targets post-discharge care; the current synthetic fixture covers heart failure (`discharge-chf-high.txt`). The system coordinates care protocols by dividing responsibilities between conversational engagement and structured workflow orchestration.

### Architectural Division (Planned Architecture)
* **IBM Bob (Conversational Front-End)**: Manages patient and caregiver dialogue, collects daily symptom reports, and provides empathetic interactive communication.
* **Langflow (Backend Workflow Orchestration)**: Handles structured data extraction, deterministic heuristic risk tiering, care plan drafting, human-in-the-loop clinical review gates, and grounded guideline retrieval.

> **Current Implementation Status**: The planned architecture is **NOT functioning end-to-end today**. F1 **PASSED** validation and the exact synthetic fixture run on operator-authorized `gemini-3.5-flash-lite` at temperature 0: valid JSON, score 7, HIGH, complete episode, all four factors. Session isolation was proven through the approved REST run route. F2 is authorized next but remains unstarted at this checkpoint; F3–F5 remain unstarted.

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
│   └── register_discharge_summary.json       # Verbatim redacted Langflow export for F1 (4 nodes, 3 edges)
├── fixtures/
│   └── discharge-chf-high.txt                # Synthetic Indonesian CHF discharge summary text (357 bytes)
├── protocols/
│   ├── hf-aftercare.txt                      # Generic CHF post-discharge educational draft (429 words)
│   └── medication-safety.txt                 # Generic medication adherence and safety draft (391 words)
├── reports/
│   ├── langflow-discovery.txt                # Langflow MCP environment & component discovery report
│   ├── langflow-discovery-transcript.txt     # Raw and summarized tool schemas & discovery transcript
│   └── register_discharge_summary-test.txt   # F1 repair transcript, model-404 evidence, local scorer smoke
└── README.md                                 # Full agent handoff documentation
```

### Educational Protocol Drafts & Audit Artifacts
* [`protocols/hf-aftercare.txt`](protocols/hf-aftercare.txt) (429 words): Generic Indonesian post-discharge education draft covering activity guidelines, sodium restriction, daily weight tracking, and hospital contact. Strictly pending clinician review.
* [`protocols/medication-safety.txt`](protocols/medication-safety.txt) (391 words): Generic Indonesian medication adherence draft emphasizing schedule consistency, safe storage, and strict prohibition of unapproved dosage changes. Strictly pending clinician review.
* [`reports/langflow-discovery.txt`](reports/langflow-discovery.txt): Environment discovery report documenting connected Langflow MCP tools, component registry, and inventory of 35 existing server flows.
* [`reports/langflow-discovery-transcript.txt`](reports/langflow-discovery-transcript.txt): Tool schema extractions and discovery transcript.
* [`reports/register_discharge_summary-test.txt`](reports/register_discharge_summary-test.txt): Factual F1 repair transcript, verbatim mutation/validation results, provider-404 trace, reference-name evidence, and explicitly local scorer smoke outputs.
* [`exports/register_discharge_summary.json`](exports/register_discharge_summary.json): Exact raw export of F1 flow graph (nodes, edges, config, and source).

---

## 3. Five-Flow Pipeline Architecture & Implementation Status

| Flow ID | Flow Name | Purpose & Scope | Implementation Status |
|---|---|---|---|
| **F1** | `register_discharge_summary` | Ingests discharge summary text, extracts clinical episode parameters via LLM, and calculates a deterministic DEMO risk tier (LOW / MEDIUM / HIGH). **NOT a calibrated 30-day prediction model**. | **PASSED — fixture 7/HIGH; isolated REST session** |
| **F2** | `record_patient_checkin` | Ingests daily check-in text, detects 5 red-flag categories (immediate escalation), tracks non-flag symptom trajectory / adherence, calculates next check-in interval, and triggers plan review on tier changes. | **UNSTARTED — authorized after F1 acceptance** |
| **F3** | `draft_followup_plan` | Generates a JSON draft follow-up care plan in Bahasa Indonesia plus SATUSEHAT-shaped FHIR resources under key `satuseshat` (`careplan`, `servicerequest`, `task`). | **UNSTARTED** |
| **F4** | `review_care_plan` | Human-in-the-loop review via `HumanInput`, SQLite persistence (`approved_plans`, `audit_log`), and plan routing (`Approve`, `Request Changes`, `Escalate`). | **UNSTARTED** |
| **F5** | `answer_care_questions` | Grounded patient Q&A over APPROVED care plan and protocol chunks via Chroma vector retrieval with exact embedding model `models/gemini-embedding-001`, strict out-of-scope refusal, and red-flag bypass. | **UNSTARTED** |

> **Flow ID Notice**: Current instance F1 ID is `458d7c88-812b-4cb2-bc3f-ccccf809ed1a`. It pre-existed this repair and was updated in place; no flow was recreated or duplicated. Original-instance ID `e1267720-b42b-4bcf-aff5-1792a4871df1` is absent from the current inventory. Unrelated flows were not modified.

---

## 4. F1 (`register_discharge_summary`) — Passed Fixture Acceptance & Repair History

### 4.1 Expected Pipeline & Deterministic Scoring
1. **Input**: Synthetic Indonesian discharge summary text (`fixtures/discharge-chf-high.txt`: 62M, CHF NYHA III, LOS 6 days, emergency admission via IGD, DM2 + Hypertension + CKD stage 3 [3 comorbidities], 2 admissions in past 12m).
2. **LLM Extraction**: Exact type `ext:google:GoogleGenerativeAIComponent@official`, model `gemini-3.5-flash-lite` per operator decision **2026-10-03**, explicit `temperature: 0`. It is absent from the registry's static option list, but `model_name.combobox: true` permits this custom value; inspection confirmed it exactly and real execution succeeded.
   * **Episode schema**: Exactly `diagnosis` (string), `medications`, `instructions`, `followup_needs` (arrays of stated strings), `length_of_stay_days`, `comorbidity_count`, `admissions_prior_12m` (nonnegative integer or `null`), and `admission_type` (`emergency`, `elective`, or `unknown`). IGD/emergency maps to `emergency`; absent facts remain empty/null/unknown, never invented.
   * **Credential binding proven**: Read-only GET of this flow confirms `api_key.value: "GEMINI_API_KEY"` and `load_from_db: true`; successful current Gemini execution establishes runtime binding. This is a variable NAME, not a credential value; no global variables were read or changed. MCP inspection/export redacts this reference, which was left untouched.
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

### 4.2 Passed Validation & Exact Fixture Run (2026-10-03)
`validate_flow` returned `{"valid": true, "component_count": 4, "errors": []}`. Both the MCP fixture run and approved session-corrective REST fixture run produced `json.loads`-valid ChatOutput text with score **7**, tier **HIGH**, all four factors met, full eight-field episode, rationale, `demo-heuristic-v1`, and `missing_fields: []`. Medication strings and stated instructions/follow-up were preserved. Actual responses are captured verbatim in the appended transcript section.

MCP component session tweaks did **not** isolate the run: response, message, and trace used the flow UUID. The operator-approved REST `POST /api/v1/run/{flow_id}` with top-level `session_id: "demo-chf-high"` plus the same component tweaks isolated all three observed IDs to `demo-chf-high`. No credential headers were printed or persisted.

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

## 5. Specification for Remaining Flows (F2–F5) & Operator Stop Phrases

### 5.1 Operating Discipline & Multi-Agent Stop Phrases
* **Per-Flow Lifecycle**: discover $\rightarrow$ one-shot create $\rightarrow$ validate $\rightarrow$ real run $\rightarrow$ `notify_done` $\rightarrow$ redacted export + actual transcript $\rightarrow$ atomic commit.
* **Error Discipline**: Two identical consecutive errors $\rightarrow$ **STOP immediately**.
* **Safety Mandates**: Preserve other user flows on the server; never store or commit literal API secrets.
* **Authorized Scope**: Operator authorized F1/F2 model `gemini-3.5-flash-lite` at temperature 0 on 2026-10-03. Complete F2 only after F1 passes, then STOP; F3 must not begin.

**Exact Stop & Resume Phrases (Do NOT invent dialogue)**:
1. **STOP1** (after completing F1 and F2): Resume upon operator command:
   ```text
   checkpoint-1-mvp-intake selesai, lanjut
   ```
2. **STOP2** (after completing F3 and F4): Resume upon operator command:
   ```text
   checkpoint-2-human-gate selesai, lanjut
   ```
3. **STOP3** (after completing F5, inventory verification, and exporting all 5 flows): **Halt and await further operator instructions; create nothing new**.

---
### 5.2 Handoff Next Actions (Immediate Sequence for Incoming Agent)
1. **F1 Accepted**: Preserve model `gemini-3.5-flash-lite`, temperature 0, eight-field extraction/scorer contract, and existing named credential reference. The old `gemini-2.5-flash` 404 is resolved history, not a current blocker.
2. **Authorized F2 Work**: Use the same model at temperature 0, structural red-flag bypass, deterministic baseline scoring, and all three synthetic acceptance cases from section 5.3.
3. **Sessions & Audit**: Use the approved REST `/api/v1/run/{flow_id}` with explicit top-level `session_id: "demo-<patient_id>"`; component tweaks alone are ineffective on this instance. Verify message and trace IDs, capture actual outputs/bypass evidence, export verbatim, and commit atomically.
4. **Stop Gate**: STOP after F2 and await operator instruction. Never start F3 in this workstream; model choices for F3/F5 remain pending.

---


### 5.3 Flow F2: `record_patient_checkin` Specifications
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

---

### 5.4 Flow F3: `draft_followup_plan` Specifications
* **LLM Engine**: Gemini `gemini-2.5-flash`, `temperature: 0.2`.
* **Model choice pending**: The 2026-10-03 replacement decision applies only to F1/F2; this unstarted F3 specification is retained pending operator confirmation.
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

---

### 5.5 Flow F4: `review_care_plan` Specifications
* **Human-in-the-Loop Component**: `HumanInput` (confirmed PRESENT on instance).
* **Configured Actions**: `Approve`, `Request Changes`, `Escalate`, fallback `Escalate`, timeout 300s (custom action strings and timeout currently untested on this runtime).
* **Action Logic**:
  * `Approve`: Writes approved plan record to SQLite table `approved_plans(plan_json, approved_at, reviewer_note)` + outputs confirmation message in Bahasa Indonesia.
  * `Request Changes`: Loops back to plan drafting with reviewer critique notes.
  * `Escalate`: Emits escalation payload: `{"action": "escalate_to_clinician", "plan_json": {...}}`.
* **Audit Trail**: All three actions append rows to SQLite table `audit_log(action, actor, timestamp, payload)`.
* **Integrity Invariant**: **NOTHING** is written to `approved_plans` prior to explicit human approval.
* **Acceptance Tests**: Test pause event verbatim, verify `Approve` with SQLite persistence; test second scenario triggering `Escalate`.

---

### 5.6 Flow F5: `answer_care_questions` Specifications
* **Vector Store & Embeddings**: Local Chroma vector store with EXACT embedding model `models/gemini-embedding-001` (no alternate or fallback embedding models).
* **Retrieval Scope**: Top-4 retrieved chunks scoped strictly to the patient's APPROVED care plan and protocol documents.
* **Generator**: Gemini `gemini-2.5-flash`, `temperature: 0.2` generating answers in Bahasa Indonesia with citations to source chunks.
* **Model choice pending**: The 2026-10-03 replacement decision applies only to F1/F2; this unstarted F5 specification is retained pending operator confirmation.
* **Out-of-Scope Refusal Guard**: If query falls outside retrieved context, output EXACT refusal string:
  ```text
  Maaf, pertanyaan itu di luar cakupan rencana Anda. Silakan tanyakan ke perawat Anda atau hubungi hotline.
  ```
* **Red-Flag Bypass**: If query expresses red-flag symptoms, bypass LLM generation and immediately output escalation action.
* **Acceptance Tests**:
  1. In-scope lifestyle/diet question answered with source citation.
  2. Out-of-scope question (`"apakah saya kanker?"`) returns exact refusal string.
  3. Red-flag query triggers immediate escalation.

---

## 6. Manual Setup, Operator URL & Local Verification

### 6.1 Langflow Server URL & Flow Import
* **Server URL**: The configured MCP connection for the current Windows instance was discovered as `http://127.0.0.1:7860`; this is an observed instance setting, not a portable default. Existing credentials were used without printing or writing credential values.
* **Import Protection**: Check list_flows first. Import via UI only if named flow is absent; otherwise inspect/update existing flow; importer duplicate behavior is unverified.

### 6.2 Local Artifact Smoke Test (No Workflow Execution)
To verify the structural integrity of the exported flow JSON:
```bash
python3 -c "import json; d=json.load(open('exports/register_discharge_summary.json')); print('Flow Name:', d.get('name'), '| Nodes:', len(d['data']['nodes']), '| Edges:', len(d['data']['edges']))"
```
*(Expected output: `Flow Name: register_discharge_summary | Nodes: 4 | Edges: 3`)*

> **Note**: This is a local file structure smoke test only, **NOT a workflow execution test**.

### 6.3 MCP `run_flow` Schema Limitation
The MCP schema has no top-level `session_id`. Actual `run_flow` with ChatInput/ChatOutput `session_id: "demo-chf-high"` tweaks returned the **flow UUID** in its response, output message, and trace: the API's effective graph session overrides component tweaks. The operator approved direct REST `POST /api/v1/run/{flow_id}` with top-level `session_id` and the same tweaks. F1's corrective run confirmed response/message/trace IDs all equal `demo-chf-high`. Use this REST route for all F2 `demo-<patient_id>` tests. Existing configured credentials are used privately; transcript headers are redacted. Message storage remains disabled on F1.

---

## 7. Deliverables & Screenshot Capture Plan

### Stage 1 (Current Actual State)
1. Screenshot of the actual F1 build compilation error in the Langflow UI (opaque "Build error").
2. Screenshot of `CustomComponent-UW8Uq` in the Langflow UI showing the stale `build_output` template method and unrefreshed parameters.

### Subsequent Stages (Post-Repair & Future Flows)
1. F1 successful build graph and Chat Playground trace with fixture text.
2. F2 red-flag routing branch and non-flag scoring run.
3. F3 SATUSEHAT (satuseshat) JSON output structure.
4. F4 `HumanInput` review modal and SQLite database write confirmation.
5. F5 Q&A answer with source citation, out-of-context refusal, and red-flag escalation.
