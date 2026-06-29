# Failure Analysis: Case 2/5 - Multi-Currency Ambiguity

**Document ID**: TEST-PO-009 (analyzed)  
**Root Cause**: Multi-currency document with ambiguous total specification  
**Severity**: LOW (valid JSON produced despite ambiguity)  

The model extracted EUR total instead of USD consolidation. This is fundamentally an input design problem - real documents should specify single currency. Fine-tuning data (0 multi-currency examples) couldn't address this edge case.

**Fix**: Add 3-5 multi-currency examples to training set, or implement pre-validation to flag ambiguous documents

```
[Full raw document text]
```

---

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
