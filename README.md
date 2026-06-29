# Structured Output Fine-Tuning: Llama 3.2 for JSON Extraction

## Project Overview

This project fine-tunes Llama 3.2 (3B-Instruct) to reliably extract structured JSON data from unstructured business documents (invoices and purchase orders) using LlamaFactory's web UI and LoRA parameter-efficient fine-tuning. The core objective is to maximize **parse success rate** — the percentage of model responses that are valid, parseable JSON with all required keys.

### Why This Matters

General-purpose LLMs struggle with output consistency:
- Sometimes wrap JSON in markdown code fences
- Sometimes add explanatory prose before/after the JSON
- Sometimes invent fields not present in the source
- Sometimes return inconsistent key structures

Fine-tuning on curated structured examples trains the model to treat JSON formatting as a **hard constraint**, not a stylistic preference. Production pipelines require 95%+ parse success rate — accuracy is secondary if the output isn't machine-parseable.

## Project Structure

```
.
├── schema/                          # JSON schema definitions (BINDING)
│   ├── invoice_schema.md           # Invoice field definitions
│   └── po_schema.md                # Purchase order field definitions
├── data/                           # Training data
│   ├── curated_train.jsonl        # 80 curated training examples
│   └── curation_log.md            # Audit trail of data curation decisions
├── training_config.md              # Hyperparameter justification
├── screenshots/                    # LlamaFactory configuration & loss curve
│   ├── training_config.png
│   └── loss_curve.png
├── eval/                           # Evaluation artifacts
│   ├── baseline_responses.md       # Raw pre-tuning model outputs (20 docs)
│   ├── baseline_scores.csv         # Baseline scoring matrix
│   ├── finetuned_responses.md      # Raw post-tuning model outputs (20 docs)
│   ├── finetuned_scores.csv        # Post-tuning scoring matrix
│   ├── before_vs_after.md          # Side-by-side comparison table
│   ├── summary.md                  # Baseline parse success rate
│   └── failures/                   # Failure case analysis
│       ├── failure_01.md through failure_05.md
├── prompts/                        # Prompt engineering experiment
│   ├── prompt_iterations.md        # 3+ distinct prompt versions
│   └── prompt_eval.md              # Results on 3 worst baseline docs
└── report.md                       # Final analysis: prompting vs. fine-tuning

```

## Timeline & Phases

| Phase | Task | Effort | Status |
|-------|------|--------|--------|
| **A** | Schema Design | 2-3h | ✓ DONE |
| **B** | Data Curation (80 examples) | 8-12h | ⏳ IN PROGRESS |
| **C** | Baseline Evaluation (20 heldout docs) | 2-3h | ⏳ WAITING |
| **D** | Fine-Tuning via LlamaFactory UI | 1-2h | ⏳ WAITING |
| **E** | Post Fine-Tuning Evaluation | 1-2h | ⏳ WAITING |
| **F** | Failure Analysis (5 cases) | 2-3h | ⏳ WAITING |
| **G** | Prompt Engineering Comparison | 2-3h | ⏳ WAITING |
| **TOTAL** | | ~20-28h | |

---

## Phase Details & Deliverables

### Phase A: Schema Design ✓ COMPLETE

**Deliverables:**
- [schema/invoice_schema.md](schema/invoice_schema.md) — All required keys, types, constraints, handling of absent fields
- [schema/po_schema.md](schema/po_schema.md) — PO schema with consistent structure

**Key Rules:**
- Every training example must match the schema exactly
- All floats must be JSON numbers (not strings)
- All dates in ISO 8601 format (`YYYY-MM-DD`)
- `due_date` and `tax` (invoices) and `delivery_date` (POs) may be `null`
- All other fields must be present and valid

---

### Phase B: Data Curation (80 Examples) ⏳ TODO

**Objective:** Curate 50 invoices + 30 purchase orders from public datasets, manually verify ground truth JSON for each.

**Datasets:**
- **CORD v2** (Clova OCR Receipt Dataset): https://huggingface.co/datasets/naver-clova-ix/cord-v2
- **SROIE**: https://huggingface.co/datasets/AdamCodd/sroie2019
- **DocVQA**: https://huggingface.co/datasets/nielsr/docvqa_1200_examples
- **Synthetic POs**: https://huggingface.co/datasets/katanaml-org/invoices-donut-data-v1

**Curation Rules (MANDATORY):**
1. Every example must have **unique document layout** (different column orders, formatting, styling)
2. **15+ examples** with absent optional fields (null `due_date`, null `tax`, etc.)
3. **10+ examples** with 3+ line items (multi-item invoices/POs)
4. **5+ examples** with non-USD currency (GBP, EUR, INR, JPY)
5. Reject any example with uncertain ground truth
6. **Log every decision** in `data/curation_log.md`

**Deliverables:**
- [data/curated_train.jsonl](data/curated_train.jsonl) — 80 lines, each a valid training example
- [data/curation_log.md](data/curation_log.md) — Audit trail of all curation decisions

**JSONL Format:**
```jsonl
{"instruction": "Extract all invoice fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "<raw document text>", "output": "<valid JSON matching schema>"}
{"instruction": "Extract all purchase order fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "<raw PO text>", "output": "<valid JSON matching schema>"}
```

---

### Phase C: Baseline Evaluation ⏳ TODO

**Objective:** Measure baseline (pre-tuning) parse success rate on 20 held-out documents.

**Steps:**
1. Download 20 heldout documents from datasets (must be different from training set)
2. Design one extraction prompt (best you can write, no fine-tuning yet)
3. Run prompt against all 20 docs using LlamaFactory inference tab or Open WebUI with `llama3.2:3b`
4. Copy every raw response verbatim
5. Score each response:
   - `is_valid_json`: Can Python's `json.loads()` parse it? (test: `python3 -c "import json; json.loads(open('response.txt').read())"`)
   - `has_all_required_keys`: Are all schema keys present?
   - `key_accuracy`: Fraction of required keys present
   - `value_accuracy`: Of present values, what fraction match ground truth?
6. Compute **baseline parse success rate** = `(count where is_valid_json AND has_all_required_keys) / 20`

**Deliverables:**
- [eval/baseline_responses.md](eval/baseline_responses.md) — Verbatim raw outputs for 20 docs
- [eval/baseline_scores.csv](eval/baseline_scores.csv) — Scoring matrix
- [eval/summary.md](eval/summary.md) — Baseline parse success rate (prominently stated)

**Example baseline_scores.csv:**
```csv
document,raw_output_first_50_chars,is_valid_json,has_all_required_keys,key_accuracy,value_accuracy,notes
invoice_01.txt,"Here's the extracted data: {",False,False,0.5,0.3,"Wrapped in code fences"
...
```

---

### Phase D: Fine-Tuning via LlamaFactory UI ⏳ TODO

**Steps:**
1. Install LlamaFactory: `pip install llamafactory`
2. Launch web UI: `llamafactory-cli webui`
3. Navigate to **Fine-Tune** tab
4. Configure:
   - **Model**: Llama-3.2-3B-Instruct
   - **Dataset**: `data/curated_train.jsonl`
   - **Method**: LoRA
   - **LoRA Rank**: 8, 16, or 32 (justify choice)
   - **LoRA Alpha**: 2× rank (e.g., 32 for rank 16)
   - **Learning Rate**: 1e-4 to 3e-4 (justify)
   - **Epochs**: 2–5 (justify based on dataset size & overfitting risk)
   - **Batch Size**: Maximum that fits in your RAM
5. Screenshot configuration panel
6. Monitor loss curve during training
7. Screenshot loss curve at completion

**Deliverables:**
- [training_config.md](training_config.md) — Detailed hyperparameter justification + loss curve observations
- [screenshots/training_config.png](screenshots/training_config.png) — Config panel before training
- [screenshots/loss_curve.png](screenshots/loss_curve.png) — Loss curve after training

**training_config.md Template:**
```markdown
# Training Configuration & Justification

## Hyperparameter Choices

### LoRA Rank
- **Choice**: [8/16/32]
- **Justification**: [Why this value for your task complexity & dataset size?]

### LoRA Alpha
- **Choice**: [2×rank]
- **Justification**: Standard practice; controls scaling of LoRA updates

### Learning Rate
- **Choice**: [specific value]
- **Justification**: [Balance between convergence speed and stability?]

### Epochs
- **Choice**: [number]
- **Justification**: [Based on dataset size and overfitting prevention?]

### Batch Size
- **Choice**: [size]
- **Justification**: [Hardware constraints?]

## Training Observations

[Describe the loss curve: steady descent? plateau? overfitting? Why?]
[If multiple runs, document all hyperparameter changes and reasoning]
```

---

### Phase E: Post Fine-Tuning Evaluation ⏳ TODO

**Steps:**
1. Load fine-tuned model in LlamaFactory inference tab
2. Run **exact same 20 held-out documents** from Phase C with **exact same prompt**
3. Copy all raw responses verbatim
4. Score using same CSV format as Phase C
5. Compute **post-tuning parse success rate**
6. Build side-by-side comparison table

**Deliverables:**
- [eval/finetuned_responses.md](eval/finetuned_responses.md) — Raw responses (20 docs)
- [eval/finetuned_scores.csv](eval/finetuned_scores.csv) — Scoring matrix
- [eval/before_vs_after.md](eval/before_vs_after.md) — Comparison table

**before_vs_after.md Template:**
```markdown
# Baseline vs. Fine-Tuned: Side-by-Side Comparison

| Metric | Baseline (Base Model) | Post Fine-Tuning | Improvement |
|--------|----------------------|------------------|-------------|
| Parse Success Rate | X% | Y% | +Z% |
| Avg Key Accuracy | | | |
| Avg Value Accuracy | | | |
| Responses with Markdown Fences | | | |
| Responses with Prose Preamble | | | |
| Responses with Wrong Schema Keys | | | |

## Key Findings

[What changed most? Why?]
[Failure patterns that remained?]
```

---

### Phase F: Failure Analysis (5 Cases) ⏳ TODO

**Objective:** Diagnose why fine-tuned model still fails on 5 difficult examples.

**For each of 5 failures:**
1. Document: raw document text
2. Expected: correct JSON per schema
3. Actual: model's incorrect response
4. Failure mode: formatting issue? key naming? hallucination?
5. Root cause: underrepresented in training? unusual layout? ambiguous field?
6. Data fix proposal: specific JSONL addition/modification (not a prompt change)

**Deliverables:**
- [eval/failures/failure_01.md](eval/failures/failure_01.md) through [failure_05.md](eval/failures/failure_05.md)

**Template (failure_01.md):**
```markdown
# Failure Case 01

## Source Document
[Raw invoice/PO text]

## Expected JSON Output
[Correct JSON per schema]

## Model's Actual Output
[Verbatim incorrect response]

## Failure Analysis

### Failure Mode
[Describe the exact failure: invalid JSON? Missing keys? Type mismatch? Hallucinated fields?]

### Why It Likely Failed
[Data underrepresentation? Unusual layout? Ambiguous field interpretation?]

### Proposed Data Fix
[Specific new training example or modification to curated_train.jsonl that would prevent this failure]
```

---

### Phase G: Prompt Engineering Comparison ⏳ TODO

**Objective:** Compare prompt engineering vs. fine-tuning on 3 worst baseline documents.

**Steps:**
1. Identify 3 documents with lowest scores from Phase C (worst performers)
2. Design 3+ distinct prompt versions to improve base model output:
   - Prompt v1: Few-shot examples
   - Prompt v2: Stricter format instructions
   - Prompt v3: Explicit "do not use markdown" constraints
3. Test each prompt against all 3 documents
4. Record all prompts and results

**Deliverables:**
- [prompts/prompt_iterations.md](prompts/prompt_iterations.md) — 3+ prompt versions
- [prompts/prompt_eval.md](prompts/prompt_eval.md) — Results comparison
- [report.md](report.md) — ~300-word analysis

**report.md Section: "Prompting vs. Fine-Tuning"**
```markdown
# Prompting vs. Fine-Tuning: Analysis & Recommendations

## Findings

[Baseline prompt success rate on 3 worst docs: X%]
[Best prompt-only success rate: Y%]
[Fine-tuned model success rate: Z%]

## When Prompt Engineering Wins
[Conditions where iterative prompting was more effective?]

## When Fine-Tuning Wins
[Conditions where fine-tuning dominated?]

## Production Recommendation
[When would you choose prompt engineering? Fine-tuning? Hybrid?]
[What does this reveal about the reliability problem in enterprise AI?]

[~300 words total]
```

---

## Installation & Setup

### Requirements
- Python 3.10+
- LlamaFactory: `pip install llamafactory`
- GPU recommended (Llama 3.2 3B can run on CPU, but training will be slow)
- 8GB+ RAM for LoRA fine-tuning

### Quick Start

```bash
# 1. Install LlamaFactory
pip install llamafactory

# 2. Launch web UI
llamafactory-cli webui

# 3. Open browser to http://127.0.0.1:7860

# 4. Follow Phase B (data curation), then Phase D (fine-tuning)
```

---

## Evaluation Criteria

Your submission will be assessed on:

1. **Schema Quality**: Are the schemas complete, consistent, and do all training examples strictly comply?
2. **Data Curation Rigor**: Is the curation log detailed? Do examples cover required diversity (multiple layouts, nullable fields, multi-item examples, non-USD currencies)?
3. **Baseline Establishment**: Is the baseline parse success rate clearly computed and documented? Are ground truths verified?
4. **Fine-Tuning Justification**: Are all hyperparameter choices explained with clear reasoning?
5. **Before-vs-After Analysis**: Is the improvement in parse success rate statistically meaningful? Are failure modes analyzed?
6. **Failure Case Depth**: Do failure analyses reveal genuine data curation insights?
7. **Prompt Engineering Experiment**: Does the comparison reveal actionable distinctions between prompting and fine-tuning?

---

## Key Metrics

- **Baseline Parse Success Rate**: % of base model responses that are valid JSON with all required keys
- **Post-Tuning Parse Success Rate**: % of fine-tuned model responses that are valid JSON
- **Target**: 95%+ parse success rate (accuracy is secondary)

---

## References

- [LlamaFactory GitHub](https://github.com/hiyouga/LLaMA-Factory)
- [Hugging Face LoRA Guide](https://huggingface.co/docs/peft/conceptual_guides/lora)
- [JSONL Format](https://jsonlines.org/)
- [ISO 8601 Dates](https://en.wikipedia.org/wiki/ISO_8601)

---

## Next Steps

1. ✓ **Phase A (Complete)**: Review [schema/invoice_schema.md](schema/invoice_schema.md) and [schema/po_schema.md](schema/po_schema.md)
2. **Phase B (Start Now)**: Begin curating 80 training examples using the datasets listed above
3. Follow remaining phases sequentially

Good luck! This task mirrors real enterprise challenges in document automation.
