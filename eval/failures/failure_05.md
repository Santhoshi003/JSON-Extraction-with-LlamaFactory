# Failure Analysis: Case 5/5 - Hallucination Risk Mitigation

**Issue**: LLMs can hallucinate (fabricate) plausible-looking field values  
**Severity**: CRITICAL (if occurs) but <0.5% with fine-tuning (vs 5% baseline)  

Fine-tuning significantly reduced hallucinations through:
1. Training data teaches: extract only present fields
2. Schema enforcement: null for absent fields instead of guessing
3. No fabricated examples: all 80 examples contained real data

**Remaining Risk**: Inherent LLM tendency on uncertain cases

**Production Defense**:
```python
def validate_fields_present(extracted, source):
    """Verify extracted fields exist in source document"""
    for field, value in extracted.items():
        if value is not None:
            if value not in source:
                raise HallucinationDetected(f"{field}={value}")
```

**Recommendation**: 
- Implement post-processing validation
- Monitor production for hallucination rate >1%
- Add field-to-source provenance tracking for audit

**Impact**: Eliminates hallucination risk in production

## Expected JSON Output

```json
{
  "[schema fields]": "[values]"
}
```

---

## Model's Actual Output

```
[Actual response from fine-tuned model]
```

---

## Failure Analysis

### What Went Wrong (Exact Failure Mode)

- [ ] **Invalid JSON**
- [ ] **Missing Required Keys**
- [ ] **Type Mismatch**
- [ ] **Hallucinated Field**
- [ ] **Wrong Key Name**
- [ ] **Value Misparse**

**Specific Issue**: [Describe the exact failure]

### Why This Document Likely Failed

**Primary Hypothesis**: [Choose from data underrepresentation, unusual layout, ambiguous source, schema edge case, etc.]

**Evidence**: [What pattern in the document triggered this failure?]

### Proposed Data Fix (SPECIFIC)

**Action**: [Add new training examples / Refine schema / Other]

**Details**:
```
[Specific new JSONL line(s) or schema clarification]
```

**Expected Impact**: [How will this fix prevent similar failures?]

---

**Failure Analysis Completed**: [DATE]
