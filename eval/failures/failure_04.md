# Failure Analysis: Case 4/5 - Large Number Edge Case

**Document ID**: Edge case identified during analysis  
**Root Cause**: Extreme quantity range (500K units @ $0.035)  
**Severity**: LOW  

Training data included quantity ranges 1-500,000 units and prices $0.035-$2.5M, but extreme combinations (max quantity + min price) were not explicitly trained. Model occasionally misplaces decimal on edge cases.

**Training Gap**: No explicit examples with (500K units @ $0.035 = $17,500) combinations

**Fix**: Add 2-3 edge case examples:
- 500,000 units @ $0.035/unit = $17,500
- 1 unit @ $2,500,000 = $2,500,000

**Effort**: LOW (2 examples, 10 minutes)  
**Impact**: Eliminates decimal placement errors on extremes

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
