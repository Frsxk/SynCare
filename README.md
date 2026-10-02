# SynCare — Risk-Adaptive Post-Discharge Monitoring Workflow (Hacktiv8 x IBM 2026)

## 1. Executive Summary & System Scope

**SynCare** is a risk-adaptive post-discharge patient monitoring prototype tailored for the Indonesian healthcare context, developed for the **Hacktiv8 x IBM 2026** initiative. The planned workflow targets post-discharge care; the current synthetic fixture covers heart failure (`discharge-chf-high.txt`). The system coordinates care protocols by dividing responsibilities between conversational engagement and structured workflow orchestration.

### Architectural Division (Planned Architecture)
* **IBM Bob (Conversational Front-End)**: Manages patient and caregiver dialogue, collects daily symptom reports, and provides empathetic interactive communication.
* **Langflow (Backend Workflow Orchestration)**: Handles structured data extraction, deterministic heuristic risk tiering, care plan drafting, human-in-the-loop clinical review gates, and grounded guideline retrieval.

> **Current Implementation Status**: The planned architecture is **NOT functioning end-to-end today**. Only Flow F1 exists as a non-functional draft that failed build validation. Flows F2 through F5 are completely unstarted.

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
│   └── register_discharge_summary.json       # Raw Langflow flow export for F1 (4 nodes, 3 edges, ~69 KB)
├── fixtures/
│   └── discharge-chf-high.txt                # Synthetic Indonesian CHF discharge summary text (357 bytes)
├── protocols/
│   ├── hf-aftercare.txt                      # Generic CHF post-discharge educational draft (429 words)
│   └── medication-safety.txt                 # Generic medication adherence and safety draft (391 words)
├── reports/
│   ├── langflow-discovery.txt                # Langflow MCP environment & component discovery report
│   ├── langflow-discovery-transcript.txt     # Raw and summarized tool schemas & discovery transcript
│   └── register_discharge_summary-test.txt   # F1 flow build/validation failure transcript & audit
└── README.md                                 # Full agent handoff documentation
```

### Educational Protocol Drafts & Audit Artifacts
* [`protocols/hf-aftercare.txt`](protocols/hf-aftercare.txt) (429 words): Generic Indonesian post-discharge education draft covering activity guidelines, sodium restriction, daily weight tracking, and hospital contact. Strictly pending clinician review.
* [`protocols/medication-safety.txt`](protocols/medication-safety.txt) (391 words): Generic Indonesian medication adherence draft emphasizing schedule consistency, safe storage, and strict prohibition of unapproved dosage changes. Strictly pending clinician review.
* [`reports/langflow-discovery.txt`](reports/langflow-discovery.txt): Environment discovery report documenting connected Langflow MCP tools, component registry, and inventory of 35 existing server flows.
* [`reports/langflow-discovery-transcript.txt`](reports/langflow-discovery-transcript.txt): Tool schema extractions and discovery transcript.
* [`reports/register_discharge_summary-test.txt`](reports/register_discharge_summary-test.txt): Complete audit transcript of F1 build validation failure and diagnostic findings.
* [`exports/register_discharge_summary.json`](exports/register_discharge_summary.json): Exact raw export of F1 flow graph (nodes, edges, config, and source).

---

## 3. Five-Flow Pipeline Architecture & Implementation Status

| Flow ID | Flow Name | Purpose & Scope | Implementation Status |
|---|---|---|---|
| **F1** | `register_discharge_summary` | Ingests discharge summary text, extracts clinical episode parameters via LLM, and calculates a deterministic DEMO risk tier (LOW / MEDIUM / HIGH). **NOT a calibrated 30-day prediction model**. | **DRAFT — FAILED VALIDATION (Schema Deviant)** |
| **F2** | `record_patient_checkin` | Ingests daily check-in text, detects 5 red-flag categories (immediate escalation), tracks non-flag symptom trajectory / adherence, calculates next check-in interval, and triggers plan review on tier changes. | **UNSTARTED** |
| **F3** | `draft_followup_plan` | Generates a JSON draft follow-up care plan in Bahasa Indonesia plus SATUSEHAT-shaped FHIR resources under key `satuseshat` (`careplan`, `servicerequest`, `task`). | **UNSTARTED** |
| **F4** | `review_care_plan` | Human-in-the-loop review via `HumanInput`, SQLite persistence (`approved_plans`, `audit_log`), and plan routing (`Approve`, `Request Changes`, `Escalate`). | **UNSTARTED** |
| **F5** | `answer_care_questions` | Grounded patient Q&A over APPROVED care plan and protocol chunks via Chroma vector retrieval with exact embedding model `models/gemini-embedding-001`, strict out-of-scope refusal, and red-flag bypass. | **UNSTARTED** |

> **Flow ID Notice**: F1 was created on the local server with instance ID `e1267720-b42b-4bcf-aff5-1792a4871df1`. This UUID is server-instance specific; whether re-importing the flow preserves this UUID or assigns a new one is unasserted.

---

## 4. F1 (`register_discharge_summary`) — Detailed Audit & Failure State

### 4.1 Expected Pipeline & Deterministic Scoring
1. **Input**: Synthetic Indonesian discharge summary text (`fixtures/discharge-chf-high.txt`: 62M, CHF NYHA III, LOS 6 days, emergency admission via IGD, DM2 + Hypertension + CKD stage 3 [3 comorbidities], 2 admissions in past 12m).
2. **LLM Extraction**: `GoogleGenerativeAIComponent` using model `gemini-2.5-flash` at `temperature: 0.0`.
   * **API Key Requirement**: Must bind to the Langflow global variable `GEMINI_API_KEY` (`load_from_db: true`). **NEVER stored as a literal string**.
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

### 4.2 Actual Build Validation Failure
When calling `validate_flow`, the build failed identically on two consecutive attempts:

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

* **Build Trace**: Only `ChatInput-vk8pT` recorded a valid build timestamp (`builds: {"ChatInput-vk8pT": {"valid": true}}`). The exact point of failure during graph compilation is unknown; do **NOT** assert that downstream graph compilation is the proven cause.
* **Stop Triggered**: Per the project two-failure rule, execution stopped immediately.
* **Execution Record**: `run_flow` was **NEVER run**. No session isolation was exercised.

### 4.3 Diagnostic Findings & Observed Inconsistencies
1. **Stale CustomComponent Output Method**:
   Inspection via `get_component_info` showed that `CustomComponent-UW8Uq` retained default template output properties:
   * Output method remained `"build_output"` instead of `"score_risk"`.
   * Leftover template param `"input_value": "Hello, World!"` remained unrefreshed.
2. **Import Namespace Mismatch Hypothesis**:
   * The server's native components all reside in the `lfx` namespace (`lfx` version 1.12.2, e.g., `lfx.components.custom_component.custom_component.CustomComponent`).
   * The custom scorer script imported from `langflow` (`from langflow.custom import Component`).
   * `[INFERENCE]` Package availability of `"langflow"` on this server runtime is unverified; whether the `lfx` vs `langflow` import discrepancy is the root cause of the build failure is a candidate hypothesis, **NOT a proven root cause**. Incoming agents must discover the actual supported API on the server before replacing imports; no corrective command is proven.
3. **API Key Global Binding Status**:
   * In the exported JSON, `api_key` has `load_from_db: true` and `value: ""`.
   * Having an empty field and `load_from_db: true` **DOES NOT prove that the named `GEMINI_API_KEY` global is actually bound**. Verification requires inspecting confirmed global reference metadata and exercising a successful Gemini execution.
   * Do **NOT** rename or edit existing global variables.
4. **Extraction Prompt Schema Deviations**:
   The prompt stored on the Gemini node (`system_message`) deviated significantly from the requested episode schema:
   * **Requested Contract**: `diagnosis`, `medications`, `instructions`, `followup_needs`, `length_of_stay_days`, `admission_type`, `comorbidity_count`, `admissions_prior_12m`.
   * **Actual Stored Prompt (Summary)**: Asked only for `length_of_stay_days`, `emergency_admission` (boolean), `comorbidities` (list), `admissions_prior_12m`, and `extraction_notes`.
   * **Missing Fields**: `diagnosis`, `medications`, `instructions`, `followup_needs`, `admission_type`, `comorbidity_count`.
   * **Downstream Consequence**: The scorer does not receive or validate the full episode payload.

---

## 5. Specification for Remaining Flows (F2–F5) & Operator Stop Phrases

### 5.1 Operating Discipline & Multi-Agent Stop Phrases
* **Per-Flow Lifecycle**: discover $\rightarrow$ one-shot create $\rightarrow$ validate $\rightarrow$ real run $\rightarrow$ `notify_done` $\rightarrow$ redacted export + actual transcript $\rightarrow$ atomic commit.
* **Error Discipline**: Two identical consecutive errors $\rightarrow$ **STOP immediately**.
* **Safety Mandates**: Preserve other user flows on the server; never store or commit literal API secrets.
* **Runtime Repair Hold**: Until the operator authorizes repair, no runtime modifications may be executed.

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
1. **F1 Schema Restoration & Scorer Migration**:
   - Restore the full requested episode extraction schema on Gemini: `diagnosis`, `medications`, `instructions`, `followup_needs`, `length_of_stay_days`, `admission_type`, `comorbidity_count`, `admissions_prior_12m`.
   - Migrate the deterministic Python scorer to consume `admission_type` and `comorbidity_count`, while retaining and passing through all extracted episode fields.
2. **Custom Component Template & Import Discovery**:
   - Discover the valid runtime import and template refresh mechanism for `CustomComponent` on this server (`lfx` 1.12.2).
3. **Verify Global Credential Binding**:
   - Verify confirmed global reference binding for `GEMINI_API_KEY` (`load_from_db: true`).
4. **Validation & Real Fixture Run**:
   - Validate flow; respect the two-failure STOP rule (no indefinite validation loops).
   - Once compilation passes, execute real run against `fixtures/discharge-chf-high.txt`.
   - Assert valid JSON output with score 7, HIGH tier, all 4 factors, and rationale.
5. **Audit Artifacts & Atomic Commit**:
   - Save actual run transcript and redacted export JSON; commit atomically.
6. **Authorization Gate**:
   - Resume F2 ONLY when explicitly authorized by operator (await STOP1 resume phrase: `checkpoint-1-mvp-intake selesai, lanjut`).

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
* **Server URL**: Use the operator's configured Langflow server URL (unknown; do **NOT** assume any hardcoded default or local host address).
* **Import Protection**: Check list_flows first. Import via UI only if named flow is absent; otherwise inspect/update existing flow; importer duplicate behavior is unverified.

### 6.2 Local Artifact Smoke Test (No Workflow Execution)
To verify the structural integrity of the exported flow JSON:
```bash
python3 -c "import json; d=json.load(open('exports/register_discharge_summary.json')); print('Flow Name:', d.get('name'), '| Nodes:', len(d['data']['nodes']), '| Edges:', len(d['data']['edges']))"
```
*(Expected output: `Flow Name: register_discharge_summary | Nodes: 4 | Edges: 3`)*

> **Note**: This is a local file structure smoke test only, **NOT a workflow execution test**.

### 6.3 MCP `run_flow` Schema Limitation
The connected Langflow MCP `run_flow` tool schema does not provide a top-level `session_id` parameter. Session identification (e.g., `demo-chf-high`) must be passed via `tweaks` targeting the advanced fields of `ChatInput` and `ChatOutput` (e.g., `{"ChatInput-vk8pT": {"session_id": "demo-chf-high"}}`). This tweak mechanism is currently untested on this server.

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
