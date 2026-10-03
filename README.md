# SynCare — Risk-Adaptive Post-Discharge Monitoring Workflow (Hacktiv8 x IBM 2026)

## 1. Executive Summary & System Scope

**SynCare** is a risk-adaptive post-discharge patient monitoring prototype tailored for the Indonesian healthcare context, developed for the **Hacktiv8 x IBM 2026** initiative. The planned workflow targets post-discharge care; the current synthetic fixture covers heart failure (`discharge-chf-high.txt`). The system coordinates care protocols by dividing responsibilities between conversational engagement and structured workflow orchestration.

### Architectural Division (Planned Architecture)
* **IBM Bob (Conversational Front-End)**: Manages patient and caregiver dialogue, collects daily symptom reports, and provides empathetic interactive communication.
* **Langflow (Backend Workflow Orchestration)**: Handles structured data extraction, deterministic heuristic risk tiering, care plan drafting, human-in-the-loop clinical review gates, and grounded guideline retrieval.

> **Current Implementation Status**: The planned architecture is **NOT functioning end-to-end today**. F1's extraction schema and deterministic scorer have been repaired in place, but acceptance is **BLOCKED**: validation reaches Gemini and the provider returns `404 NOT_FOUND` for required model `gemini-2.5-flash`. No repaired-flow `run_flow` was performed. Flows F2 through F5 remain unstarted; F2 shares the same model requirement.

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
| **F1** | `register_discharge_summary` | Ingests discharge summary text, extracts clinical episode parameters via LLM, and calculates a deterministic DEMO risk tier (LOW / MEDIUM / HIGH). **NOT a calibrated 30-day prediction model**. | **STRUCTURE REPAIRED — VALIDATION BLOCKED (Gemini model 404)** |
| **F2** | `record_patient_checkin` | Ingests daily check-in text, detects 5 red-flag categories (immediate escalation), tracks non-flag symptom trajectory / adherence, calculates next check-in interval, and triggers plan review on tier changes. | **UNSTARTED — shares model blocker** |
| **F3** | `draft_followup_plan` | Generates a JSON draft follow-up care plan in Bahasa Indonesia plus SATUSEHAT-shaped FHIR resources under key `satuseshat` (`careplan`, `servicerequest`, `task`). | **UNSTARTED** |
| **F4** | `review_care_plan` | Human-in-the-loop review via `HumanInput`, SQLite persistence (`approved_plans`, `audit_log`), and plan routing (`Approve`, `Request Changes`, `Escalate`). | **UNSTARTED** |
| **F5** | `answer_care_questions` | Grounded patient Q&A over APPROVED care plan and protocol chunks via Chroma vector retrieval with exact embedding model `models/gemini-embedding-001`, strict out-of-scope refusal, and red-flag bypass. | **UNSTARTED** |

> **Flow ID Notice**: Current instance F1 ID is `458d7c88-812b-4cb2-bc3f-ccccf809ed1a`. It pre-existed this repair and was updated in place; no flow was recreated or duplicated. Original-instance ID `e1267720-b42b-4bcf-aff5-1792a4871df1` is absent from the current inventory. Unrelated flows were not modified.

---

## 4. F1 (`register_discharge_summary`) — Repair Audit & Model Availability Blocker

### 4.1 Expected Pipeline & Deterministic Scoring
1. **Input**: Synthetic Indonesian discharge summary text (`fixtures/discharge-chf-high.txt`: 62M, CHF NYHA III, LOS 6 days, emergency admission via IGD, DM2 + Hypertension + CKD stage 3 [3 comorbidities], 2 admissions in past 12m).
2. **LLM Extraction**: Exact type `ext:google:GoogleGenerativeAIComponent@official`, model `gemini-2.5-flash`, explicit `temperature: 0`.
   * **Episode schema**: Exactly `diagnosis` (string), `medications`, `instructions`, `followup_needs` (arrays of stated strings), `length_of_stay_days`, `comorbidity_count`, `admissions_prior_12m` (nonnegative integer or `null`), and `admission_type` (`emergency`, `elective`, or `unknown`). IGD/emergency maps to `emergency`; absent facts remain empty/null/unknown, never invented.
   * **Credential reference**: Read-only GET of this flow confirms `api_key.value: "GEMINI_API_KEY"` and `load_from_db: true`. This is a variable NAME, not a credential value; no global variables were read or changed. MCP inspection/export redacts this non-empty reference. Binding was left untouched; a successful current Gemini execution is still unobserved.
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

### 4.2 Current Validation Blocker (2026-10-03)
One `validate_flow` call returned:

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

Read-only REST trace discovery isolated the actual current cause. Trace `9441f3c0-6502-41cd-8b2e-71349aeecac7`, timestamp `2026-10-03T12:18:04.560578`, records:

```text
Error calling model 'gemini-2.5-flash' (NOT_FOUND): 404 NOT_FOUND. {'error': {'code': 404, 'message': 'This model models/gemini-2.5-flash is no longer available to new users. Please update your code to use models/gemini-3.8-flash for the latest features and improvements. We recommend you to use the Interactions API (https://ai.google.dev/gemini-api/docs/get-started).', 'status': 'NOT_FOUND'}}
```

Only ChatInput has a fresh build record from this validation; downstream successful records are stale from the preceding day and are **not** repair acceptance evidence. No second validation or `run_flow` was attempted. The required model and temperature remain unchanged; no substitute model was selected. Operator authorization/model availability is the prerequisite to continue. `[INFERENCE]` A model-specific 404 rather than 401/403 suggests authentication succeeded, but is not a successful Gemini run.

### 4.3 Implemented Structure & Limited Verification
* Current runtime/native export identifies `lfx` **1.12.4**, not the prior instance's 1.12.2. The pre-existing current scorer already used `lfx.custom`, `lfx.io`, and `lfx.schema.message`; these supported imports were retained.
* The scorer strips JSON markdown fences, requires exactly all eight episode keys, validates value types, permits null/unknown facts, and retains the complete `episode`. It emits `score`, `tier`, all four `risk_factors` (value/threshold/points/met), `rationale`, `label: "demo-heuristic-v1"`, `missing_fields`, and synthetic/not-clinically-validated disclaimers via `Message(text=json.dumps(...))`.
* `get_component_info` confirms registered `output.method: "score_risk"` and `input_value: ""`; no `"Hello, World!"` parameter remains. Existing inputs, outputs, and edge handles were preserved. Installed MCP source shows code configuration alone does **not** re-derive the template; full custom-code template derivation is available through `/api/v1/custom_component`. No registration mismatch existed on this instance.
* ChatOutput's `clean_data` is explicitly false. Native `safe_convert` returns Message text directly; Data serialization can instead introduce JSON markdown fences. Final repaired ChatOutput behavior is **not yet exercised end-to-end**.
* Authorized throwaway local checks: exact scorer code compiled and imported/instantiated with the installed server environment. Hand-written fixture-shaped extraction yielded **7/HIGH**, all four factors met; all-unknown extraction yielded **0/LOW**, with eight missing fields. Both fenced input cases produced JSON text parseable with `json.loads`. These are **local scorer unit smokes, not Gemini extraction or flow runs**. Temporary scripts/bytecode were deleted.
* `notify_done` acknowledged a **blocked** summary. The export is the unmodified redacted `export_flow` tool output. Full mutation results, trace response, offered model list, and local smoke output are in the transcript.

### 4.4 Historical Old-Instance Failure (Unprovable Here)
On original ID `e1267720-b42b-4bcf-aff5-1792a4871df1`, two validations returned the same opaque `"Build error"` and only ChatInput recorded a build. The old export stored `langflow.*` scorer imports while native sources used `lfx` 1.12.2, stale method `build_output`, `"Hello, World!"`, empty API reference, and a deviating extraction schema. These are historical observations; the exact old failure cause remains unproven. The current model-404 trace does **not** establish the root cause on that absent instance.

---

## 5. Specification for Remaining Flows (F2–F5) & Operator Stop Phrases

### 5.1 Operating Discipline & Multi-Agent Stop Phrases
* **Per-Flow Lifecycle**: discover $\rightarrow$ one-shot create $\rightarrow$ validate $\rightarrow$ real run $\rightarrow$ `notify_done` $\rightarrow$ redacted export + actual transcript $\rightarrow$ atomic commit.
* **Error Discipline**: Two identical consecutive errors $\rightarrow$ **STOP immediately**.
* **Safety Mandates**: Preserve other user flows on the server; never store or commit literal API secrets.
* **Runtime Repair Hold**: F1 structural repair is authorized and retained. Further validation/model execution is on hold pending the operator's model decision; F2 still requires explicit authorization, and F3 must not begin.

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
1. **Resolve the Shared Model Prerequisite**:
   - Required `gemini-2.5-flash` returned an evidenced provider 404. Do not substitute a model or retry validation until the operator resolves availability or explicitly changes the requirement.
   - The component registry's offered model list is recorded verbatim in the F1 transcript; offered options alone do not prove provider availability.
2. **Resume F1 Acceptance Only After Authorization**:
   - Keep the restored schema, scorer, exact model/temperature, and existing named credential reference.
   - Validate the flow; respect the two-identical-failure stop rule.
   - Only after validation passes, run the exact fixture with `demo-chf-high` session tweaks and confirm final ChatOutput is valid JSON with score 7, HIGH, all four factors, all eight episode fields, rationale, and disclaimers.
   - Record successful Gemini execution and actual session evidence; local scorer smokes are insufficient.
3. **Audit & Authorization Gate**:
   - Update the raw export, actual transcript, and README atomically after acceptance.
   - F2 remains unstarted and shares the unresolved model requirement. Start F2 only on explicit operator instruction; never begin F3 in this workstream.

---


### 5.3 Flow F2: `record_patient_checkin` Specifications
* **Input Extraction**: Gemini `gemini-2.5-flash`, `temperature: 0.0`.
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
The MCP `run_flow` schema has no top-level `session_id`. Both current ChatInput (`ChatInput-wQRXg`) and ChatOutput (`ChatOutput-dkoVg`) already contain `session_id: "demo-chf-high"` with message storage disabled. The repaired-flow run and its session tweaks were **not exercised**, because validation is blocked. Current validation's REST trace uses the flow UUID as its session ID, so configured component session values must not be mistaken for run-level trace isolation. Future authorized acceptance should supply tweaks to both components, inspect actual message and trace session IDs, and report each separately.

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
