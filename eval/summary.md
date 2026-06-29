# Baseline Evaluation Summary

## Baseline Configuration

| Setting | Value |
|---------|-------|
| **Model** | Llama 3.2 (Base) - No fine-tuning |
| **Evaluation Set** | 20 held-out documents (not in training set) |
| **Extraction Prompt** | [Your best engineered prompt here] |
| **Evaluation Tool** | LlamaFactory Inference Tab / Open WebUI |
| **Date Evaluated** | [DATE] |

---

## Baseline Parse Success Rate

**Calculation:**
- Total responses where `is_valid_json == True` AND `has_all_required_keys == True`: [X out of 20]
- **Baseline Parse Success Rate**: **X/20 = [Y%]**

**Interpretation:**
- At baseline, [Y%] of model responses are machine-parseable with all required keys
- [Z%] of responses failed parsing (invalid JSON, missing keys, wrong structure)

---

## Baseline Failure Patterns

### Common Failure Modes:
1. **Code Fence Wrapping**: [N responses wrapped in ```json ... ```]
2. **Prose Preamble**: [N responses with explanation before JSON]
3. **Missing Required Keys**: [N responses missing critical fields]
4. **Wrong Key Names**: [N responses with schema deviation]
5. **Type Mismatches**: [N responses with incorrect value types (strings instead of floats, etc.)]
6. **Hallucinated Fields**: [N responses inventing fields not in source]

---

## Detailed Response Scores

See [eval/baseline_scores.csv](../eval/baseline_scores.csv) for row-by-row scoring.

**Aggregate Statistics:**
- Average Key Accuracy: [%]
- Average Value Accuracy (among parseable responses): [%]
- Median Key Accuracy: [%]

---

## Next Phase

**Phase C Complete.** Proceed to:
1. **Phase D**: Fine-tune on data/curated_train.jsonl using LlamaFactory UI
2. **Phase E**: Evaluate fine-tuned model on same 20 documents
3. **Phase E+**: Compare before_vs_after parse success rates

---

**Baseline Evaluation Completed**: [DATE]
**Evaluator**: [YOUR NAME]
