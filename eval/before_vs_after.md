# Before vs. After: Baseline vs. Fine-Tuned Comparison

## Side-by-Side Performance Table

| Metric | Baseline (Base Model) | Post Fine-Tuning | Improvement | % Change |
|--------|----------------------|------------------|-------------|----------|
| **Parse Success Rate** | [X%] | [Y%] | [+Z%] | [+A%] |
| **Avg Key Accuracy** | [%] | [%] | [%] | |
| **Avg Value Accuracy** | [%] | [%] | [%] | |
| **Responses with Valid JSON** | [N/20] | [N/20] | | |
| **Responses with Code Fences** | [N] | [N] | [-M] | |
| **Responses with Prose Preamble** | [N] | [N] | [-M] | |
| **Responses with Wrong Schema Keys** | [N] | [N] | [-M] | |
| **Hallucinated Fields** | [N responses] | [N responses] | [change] | |

---

## Key Findings

### Parse Success Rate Improvement

**Baseline**: [X%] of responses were valid JSON with all required keys

**Post-Tuning**: [Y%] of responses met the same criteria

**Improvement**: +[Z] percentage points ([A%] relative improvement)

**Interpretation**: 
[Describe the significance of this improvement. Is it meaningful for production? Does it meet the 95% target?]

---

### Failure Mode Reduction

#### Code Fence Wrapping
- **Before**: [N] responses wrapped in ```json ... ```
- **After**: [N] responses
- **Change**: [-M] instances (baseline behavior eliminated)

**Insight**: Fine-tuning successfully trained the model to return bare JSON without markdown wrappers.

#### Prose Preamble
- **Before**: [N] responses prefixed with explanation
- **After**: [N] responses
- **Change**: [-M] instances

**Insight**: Model learned to prioritize structured output over natural explanations.

#### Wrong Schema Keys
- **Before**: [N] responses with non-schema key names
- **After**: [N] responses
- **Change**: [-M] instances

**Insight**: LoRA fine-tuning enforced schema consistency.

---

### Accuracy Improvements

#### Key Accuracy
- **Before**: [%] (average fraction of required keys present)
- **After**: [%]
- **Change**: [+%] percentage points

#### Value Accuracy
- **Before**: [%] (average fraction of values matching ground truth)
- **After**: [%]
- **Change**: [+%] percentage points

**Note**: Value accuracy depends on model's ability to extract correct facts, not just formatting. Fine-tuning helps with consistency; improving factual accuracy requires data quality and domain-specific examples.

---

## Failure Analysis Summary

### Residual Failures (Post-Tuning)

Despite improvement, [N] responses still failed to parse successfully. Common residual failure modes:

1. **[Failure Type 1]**: [N instances]
   - Example: [Brief description]
   - Root cause: [Hypothesis]

2. **[Failure Type 2]**: [N instances]
   - Example: [Brief description]
   - Root cause: [Hypothesis]

3. **[Failure Type 3]**: [N instances]
   - Example: [Brief description]
   - Root cause: [Hypothesis]

---

## Production Readiness Assessment

### Is Fine-Tuned Model Production-Ready?

**Parse Success Rate Goal**: 95%+

**Achieved Rate**: [Y%]

**Assessment**: [YES / PARTIAL / NO]

**Reasoning**:
- [If 95%+] Model meets production reliability threshold. Residual failures (5%) are acceptable for production with error handling.
- [If 85-95%] Model shows significant improvement but falls short. Recommend: [further fine-tuning / hybrid approach / prompt engineering for edge cases].
- [If <85%] Model improvement is insufficient. Recommend: [revisit training data / adjust hyperparameters / collect more examples].

---

## Conclusion

This fine-tuning experiment demonstrates [your key learning]:

1. **What improved most**: [e.g., "Format consistency (code fence elimination)"]
2. **What remained challenging**: [e.g., "Factual accuracy on ambiguous fields"]
3. **Trade-offs observed**: [e.g., "Slightly reduced factual accuracy but vastly improved parsability"]
4. **Actionable next steps**: [e.g., "Add more multi-currency examples; curate edge cases"]

---

**Comparison Completed**: [DATE]
**Evaluator**: [YOUR NAME]
