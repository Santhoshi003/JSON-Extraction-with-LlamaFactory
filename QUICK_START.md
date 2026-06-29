# Quick Start Guide: Llama 3.2 JSON Extraction Fine-Tuning

Welcome! This guide walks you through each phase of the project in order.

---

## Phase A: Schema Design ✓ COMPLETE

**Status**: DONE - You can skip this.

**Deliverables Created**:
- `schema/invoice_schema.md` — All required fields and rules
- `schema/po_schema.md` — All required fields and rules

**What You Need to Know**:
- Every training example MUST match these schemas exactly
- No missing keys (except nullable fields)
- All numbers must be JSON numbers, not strings
- All dates in `YYYY-MM-DD` format

---

## Phase B: Data Curation (80 Examples) ⏳ START HERE

**Time Required**: 8-12 hours (manual work, cannot be rushed)

**Objective**: Manually create 80 training examples (50 invoices + 30 POs)

### Step 1: Download Datasets

Open your browser and download from Hugging Face:

1. **CORD v2** (invoices/receipts): https://huggingface.co/datasets/naver-clova-ix/cord-v2
   - Look for receipt text examples
   - Download or browse online
   
2. **SROIE** (scanned receipts): https://huggingface.co/datasets/AdamCodd/sroie2019
   - Receipt text with ground truth
   
3. **DocVQA** (diverse documents): https://huggingface.co/datasets/nielsr/docvqa_1200_examples
   - Use for layout variation
   
4. **Synthetic POs**: https://huggingface.co/datasets/katanaml-org/invoices-donut-data-v1
   - Purchase order examples

### Step 2: Create Training Examples

For each of 80 examples:

1. Take raw document text from a dataset (or write synthetic if needed)
2. Manually write the correct JSON output matching the schema
3. Verify all fields are present and correct
4. Test: `python3 -c "import json; json.loads(open('test.json').read())"`

**Format**: One JSON object per line in `data/curated_train.jsonl`

**Template**:
```json
{"instruction": "Extract all invoice fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "<raw document text>", "output": "<valid JSON per schema>"}
```

See `data/JSONL_TEMPLATE.md` for 6 complete examples.

### Step 3: Ensure Data Diversity (MANDATORY)

Your 80 examples MUST include:
- ✓ **50 invoices** + **30 purchase orders**
- ✓ **≥15 examples** with null `due_date` (invoices) or null `delivery_date` (POs)
- ✓ **≥10 examples** with null `tax` (invoices)
- ✓ **≥10 examples** with 3+ line items / items (multi-item documents)
- ✓ **≥5 examples** with non-USD currency (GBP, EUR, INR, JPY, etc.)
- ✓ **≥40 different document layouts** (varied formatting, column orders, structures)

**Failure to ensure diversity will cause the model to overfit to specific layouts.**

### Step 4: Log Every Decision

Fill in `data/curation_log.md` with columns:
```
| Example ID | Document Type | Source | Decision (KEPT/REJECTED) | Reason | Schema Issues |
```

**Example**:
```
| INV-001 | Invoice | CORD v2 - receipt_42 | KEPT | Complete, diverse layout | None |
| INV-002 | Invoice | SROIE - sro_015 | REJECTED | Date illegible; ground truth unclear | date missing |
```

### Step 5: Validate Your JSONL File

```bash
python3 << 'EOF'
import json

valid_count = 0
with open('data/curated_train.jsonl', 'r') as f:
    for i, line in enumerate(f, 1):
        try:
            obj = json.loads(line)
            # Verify structure
            assert 'instruction' in obj and 'input' in obj and 'output' in obj
            # Verify output is valid JSON
            output = json.loads(obj['output'])
            valid_count += 1
            if i % 10 == 0:
                print(f"✓ Line {i} valid")
        except Exception as e:
            print(f"✗ Line {i} ERROR: {e}")
            
print(f"\nTotal valid examples: {valid_count}/80")
assert valid_count == 80, "Must have exactly 80 valid examples!"
print("✓ All 80 examples are valid!")
EOF
```

**Deliverables**:
- ✓ `data/curated_train.jsonl` (80 lines, each valid JSON)
- ✓ `data/curation_log.md` (detailed audit trail)

---

## Phase C: Baseline Evaluation ⏳ WAITING FOR PHASE B

**Time Required**: 2-3 hours

**Objective**: Measure baseline (no fine-tuning) performance on 20 held-out documents

### Step 1: Prepare 20 Evaluation Documents

1. Select 20 documents NOT in your 80-example training set
2. Download from same datasets or use synthetic examples
3. Manually create ground-truth JSON for each (conforming to schema)
4. Save as reference files for comparison

### Step 2: Design Your Best Extraction Prompt

Write one prompt (no fine-tuning yet). Example:

```
You are an expert invoice and purchase order extraction system. 
Extract all information from the provided document and return ONLY a valid JSON object with these fields:
- vendor/supplier
- invoice_number/po_number
- date (ISO 8601: YYYY-MM-DD)
- due_date/delivery_date (ISO 8601 or null)
- currency (ISO 4217, 3-letter code)
- subtotal/total (float)
- tax (float or null)
- line_items/items (array)

Return ONLY the JSON object. No explanation, no code fences, no markdown.
```

### Step 3: Install and Launch LlamaFactory

```bash
# Install
pip install llamafactory

# Launch web UI
llamafactory-cli webui

# Open browser to http://127.0.0.1:7860
```

### Step 4: Evaluate on 20 Documents

1. Navigate to **Inference** tab in LlamaFactory
2. Load **Llama 3.2 (base model, no fine-tuning)**
3. For each of 20 documents:
   - Paste your extraction prompt + raw document text
   - Copy the raw response verbatim → `eval/baseline_responses.md`
   - Score it in `eval/baseline_scores.csv`:
     - `is_valid_json`: Can `json.loads()` parse it? (test: `python3 -c "import json; json.loads(input())"`)
     - `has_all_required_keys`: All schema keys present?
     - `key_accuracy`: Fraction of required keys present
     - `value_accuracy`: Fraction of values matching ground truth

### Step 5: Calculate Baseline Parse Success Rate

```python
import csv

valid_json_and_keys = 0
with open('eval/baseline_scores.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        if row['is_valid_json'] == 'True' and row['has_all_required_keys'] == 'True':
            valid_json_and_keys += 1

baseline_rate = valid_json_and_keys / 20
print(f"Baseline Parse Success Rate: {baseline_rate:.1%}")
```

**Document this prominently in `eval/summary.md`**

**Deliverables**:
- ✓ `eval/baseline_responses.md` (raw model outputs for 20 docs)
- ✓ `eval/baseline_scores.csv` (scoring matrix)
- ✓ `eval/summary.md` (baseline parse success rate)

---

## Phase D: Fine-Tuning via LlamaFactory UI ⏳ WAITING FOR PHASE B & C

**Time Required**: 1-2 hours (setup) + training time

**Objective**: Fine-tune Llama 3.2 using LoRA on your 80 curated examples

### Step 1: Prepare Configuration

**BEFORE** launching training, decide on hyperparameters in `training_config.md`:

| Parameter | Recommendation | Your Choice | Justification |
|-----------|-----------------|---|---|
| **LoRA Rank** | 8-16 (small dataset) | [ ] | [Why?] |
| **LoRA Alpha** | 2 × rank | [ ] | Standard practice |
| **Learning Rate** | 1e-4 to 3e-4 | [ ] | [Why?] |
| **Epochs** | 2-3 (prevent overfitting) | [ ] | [Why?] |
| **Batch Size** | 8-32 (hardware dependent) | [ ] | [Why?] |

Fill in `training_config.md` with complete justifications BEFORE training.

### Step 2: Configure in LlamaFactory

1. LlamaFactory should still be running from Phase C (or restart: `llamafactory-cli webui`)
2. Navigate to **Fine-Tune** tab
3. Configure:
   - **Model**: Llama-3.2-3B-Instruct
   - **Dataset**: `data/curated_train.jsonl`
   - **Fine-tuning Method**: LoRA
   - **LoRA Rank**: [Your choice]
   - **LoRA Alpha**: [Your choice]
   - **Learning Rate**: [Your choice]
   - **Epochs**: [Your choice]
   - **Batch Size**: [Your choice]
4. **Screenshot the full configuration panel** → save as `screenshots/training_config.png`

### Step 3: Monitor Training

1. Click **Train**
2. Watch the loss curve in real-time
3. Loss should descend smoothly; if it drops to near-zero immediately, you're overfitting
4. When training completes, **screenshot the final loss curve** → save as `screenshots/loss_curve.png`
5. Document observations in `training_config.md`:
   - Did loss decrease smoothly?
   - Any plateau or inflection points?
   - Did it drop suspiciously fast?

### Step 4: Save Fine-Tuned Weights

- LlamaFactory saves the LoRA adapter automatically
- Note the directory path (you'll use it in Phase E)
- **Do NOT commit the weights to git** (.gitignore already excludes them)

**Deliverables**:
- ✓ `training_config.md` (completed with observations)
- ✓ `screenshots/training_config.png`
- ✓ `screenshots/loss_curve.png`

---

## Phase E: Post Fine-Tuning Evaluation ⏳ WAITING FOR PHASE D

**Time Required**: 1-2 hours

**Objective**: Measure fine-tuned model performance on same 20 documents

### Step 1: Load Fine-Tuned Model

1. In LlamaFactory **Inference** tab:
   - Model: Llama-3.2-3B-Instruct
   - LoRA Adapter: [Select your fine-tuned adapter from Phase D]
2. Load model

### Step 2: Evaluate on Same 20 Documents

1. Use the **exact same prompt** from Phase C
2. For each of the 20 documents (same ones as baseline):
   - Paste prompt + raw document
   - Copy raw response → `eval/finetuned_responses.md`
   - Score in `eval/finetuned_scores.csv` (same format as baseline)

### Step 3: Compare Results

Fill in `eval/before_vs_after.md`:

| Metric | Baseline | Post-Tuning | Change |
|--------|----------|-------------|--------|
| Parse Success Rate | [X%] | [Y%] | [+Z%] |
| Avg Key Accuracy | | | |
| Avg Value Accuracy | | | |
| Code Fences Wrapped | | | |
| Prose Preambles | | | |
| Wrong Schema Keys | | | |

**Questions to answer**:
- Did parse success rate improve? By how much?
- What failure mode was eliminated most effectively?
- Are residual failures fixable with more training data?

**Deliverables**:
- ✓ `eval/finetuned_responses.md` (raw responses)
- ✓ `eval/finetuned_scores.csv` (scoring matrix)
- ✓ `eval/before_vs_after.md` (comparison table)

---

## Phase F: Failure Analysis (5 Cases) ⏳ WAITING FOR PHASE E

**Time Required**: 2-3 hours

**Objective**: Diagnose 5 documents where the fine-tuned model still failed

### Step 1: Identify Failures

From your 20 evaluation documents, find 5 where fine-tuned model failed:
- Invalid JSON?
- Missing required keys?
- Wrong value types?
- Hallucinated fields?

### Step 2: Document Each Failure

For each of failures 1-5, fill in `eval/failures/failure_0X.md`:

1. **Source Document**: Paste raw text
2. **Expected JSON**: Correct output per schema
3. **Model's Actual Output**: What the model returned
4. **Failure Analysis**:
   - What went wrong? (invalid JSON / missing keys / type mismatch / hallucination)
   - Why? (data underrepresented / unusual layout / ambiguous source / schema edge case)
   - **Data fix**: Propose specific new training examples or schema clarification (not a prompt fix)

**Example**:
```markdown
# Failure Case 01

## Source Document
```
Invoice from XYZ Corp
Amount: $5,000
```

## Expected JSON
```json
{"vendor": "XYZ Corp", "total": 5000.0, ...}
```

## Model's Actual Output
```json
{"vendor": "XYZ Corp", "total": "$5000", ...}  // Wrong type!
```

## Failure Analysis

### What Went Wrong
Type mismatch: `total` should be float `5000.0`, not string `"$5000"`

### Why
Model likely trained on text with "$" symbols; didn't learn to parse and convert to float.

### Data Fix
Add 5-10 training examples with various currency symbol formats:
- "$5,000.00" → 5000.00
- "€2,500.00" → 2500.00
- "£1,000.00" → 1000.00
```

**Deliverables**:
- ✓ `eval/failures/failure_01.md` through `failure_05.md` (5 detailed analyses)

---

## Phase G: Prompt Engineering Comparison ⏳ WAITING FOR PHASE E

**Time Required**: 2-3 hours

**Objective**: Compare prompt engineering vs. fine-tuning on 3 difficult baseline documents

### Step 1: Select 3 Worst Documents

From `eval/baseline_scores.csv`, find the 3 lowest-scoring documents.

### Step 2: Engineer 3+ Prompt Versions

Try different prompt strategies on JUST these 3 documents:

**v1: Baseline Prompt** (from Phase C)
- Test on 3 docs → record results in `prompts/prompt_eval.md`

**v2: Few-Shot Examples**
- Add 2-3 example input-output pairs to the prompt
- "Here are examples of correct extraction: ..."

**v3: Strict Format Constraints**
- Add explicit rules: "Do not use markdown. Do not add explanation. Return ONLY JSON."
- "If fields are missing, use null or empty string per the schema."

**v4+: Other ideas?**
- Chain-of-thought? ("Think step by step...")
- Constraint combinations? (v2 + v3 together)

### Step 3: Score Each Version

For each prompt version, test on 3 documents:

```csv
document,prompt_version,is_valid_json,has_all_required_keys,value_accuracy,notes
doc_A,v1,False,False,0.40,"Code fences"
doc_A,v2,True,True,0.75,"Improved"
doc_A,v3,True,True,0.80,"Best"
doc_B,v1,False,False,0.20,"Prose preamble"
...
```

Fill in `prompts/prompt_eval.md`

### Step 4: Write Final Report

Fill in `report.md` Section: **"Prompting vs. Fine-Tuning: Analysis & Recommendations"** (~300 words)

**Address**:
1. What was your best prompt engineering result on 3 docs? [X%]
2. What was fine-tuned model result on same 3? [Y%]
3. When does prompting win? [Conditions]
4. When does fine-tuning win? [Conditions]
5. What would you recommend for production? [Hybrid? Fine-tune first?]
6. What does this reveal about the reliability problem in enterprise AI?

**Deliverables**:
- ✓ `prompts/prompt_iterations.md` (3+ prompt versions)
- ✓ `prompts/prompt_eval.md` (results for 3 docs)
- ✓ `report.md` (final analysis)

---

## Final Submission

### Checklist: Have You Created Everything?

**Schema** (✓ DONE):
- [ ] `schema/invoice_schema.md`
- [ ] `schema/po_schema.md`

**Data**:
- [ ] `data/curated_train.jsonl` (80 valid examples)
- [ ] `data/curation_log.md` (audit trail)

**Training**:
- [ ] `training_config.md` (hyperparameter justification)
- [ ] `screenshots/training_config.png`
- [ ] `screenshots/loss_curve.png`

**Evaluation**:
- [ ] `eval/baseline_responses.md`
- [ ] `eval/baseline_scores.csv`
- [ ] `eval/summary.md` (baseline parse success rate)
- [ ] `eval/finetuned_responses.md`
- [ ] `eval/finetuned_scores.csv`
- [ ] `eval/before_vs_after.md`
- [ ] `eval/failures/failure_01.md` through `failure_05.md`

**Prompting vs. Fine-Tuning**:
- [ ] `prompts/prompt_iterations.md`
- [ ] `prompts/prompt_eval.md`
- [ ] `report.md` (with "Prompting vs. Fine-Tuning" section)

**Git**:
- [ ] `.gitignore` (no model weights committed)
- [ ] `README.md` (project overview)

### Push to Git

```bash
git init
git add .
git commit -m "Llama 3.2 JSON Extraction Fine-Tuning: Complete project with schemas, curated data, training configs, and evaluation results"
git remote add origin <your-repo-url>
git push -u origin main
```

### Submission Ready ✓

Your repository should be clean, well-organized, and demonstrate:
1. **Rigorous data curation** (log shows real decisions)
2. **Justifiable hyperparameters** (not guesses)
3. **Before-vs-after validation** (parse success rate improvement quantified)
4. **Deep failure analysis** (specific data-centric fixes proposed)
5. **Production-ready insights** (prompting vs. fine-tuning tradeoffs understood)

---

## Troubleshooting

### LlamaFactory Won't Start
```bash
pip install --upgrade llamafactory
llamafactory-cli webui
```

### JSONL Validation Fails
```bash
# Check a single line
python3 << 'EOF'
import json
line = '{"instruction": "...", "input": "...", "output": "{...}"}'
json.loads(line)  # If this fails, the line is invalid
EOF
```

### Loss Curve Looks Wrong
- **Drops to 0 by epoch 1**: Overfitting → reduce epochs to 1 or decrease LoRA rank
- **Never decreases**: Learning rate too high → reduce to 1e-5
- **Increases or oscillates**: Learning rate too low → increase to 1e-3

### Model Evaluation Takes Too Long
- Use `llamafactory-cli inference` in terminal mode (faster than web UI)
- Or reduce evaluation set from 20 to 10 documents for testing

---

**You're ready to begin! Start with Phase B: Data Curation.**

Good luck! 🚀
