# Failure Analysis: Case 3/5 - Date Format Normalization

**Document ID**: TEST-INV-004 (training validation)  
**Root Cause**: European date format (15/04/2024) input  
**Resolution**: ✓ FIXED - Model correctly converted to ISO 8601  

This initially appeared as a risk but fine-tuning successfully resolved it. Training data included 50 invoices with consistent ISO 8601 output (2024-MM-DD) format regardless of input format (written, European, etc.).

**Key Learning**: Training data representation shapes output format. Model implicitly learned date normalization from examples.

**No additional fix needed** - demonstrates training effectiveness

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
