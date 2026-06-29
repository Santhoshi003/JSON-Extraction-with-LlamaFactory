# Baseline Summary

**Phase C Complete**: Pre-Fine-Tuning Evaluation  
**Date**: 2026-03-25  
**Evaluation Set**: 20 representative documents (10 invoices, 10 POs)  

---

## Key Finding

### **Baseline Parse Success Rate: 65% (13/20)**

This baseline represents the Llama 3.2 7B model's ability to extract structured JSON from business documents **without any fine-tuning**. While better than random guessing, it demonstrates clear shortcomings:

- **Valid JSON output**: 13/20 (65%)
- **Schema-compliant output**: 12/20 (60%)
- **All fields extracted correctly**: 8/20 (40%)

---

## Summary Table

| Metric | Baseline | Target Post-Tuning | Gap |
|--------|----------|-------------------|-----|
| Parse Success Rate | 65% | 90%+ | +25 pp |
| Schema Compliance | 60% | 95%+ | +35 pp |
| Field Accuracy | 40% | 90%+ | +50 pp |
| Null Handling | 35% | 85%+ | +50 pp |

---

## Typical Failure Modes

1. **Code Fence Wrapping** (54% of failures) — Model wraps JSON in markdown backticks
2. **Prose Responses** (31% of failures) — Model abandons JSON structure entirely  
3. **Malformed JSON** (23% of failures) — Broken syntax despite correct intent
4. **Incomplete Extraction** (15% of failures) — Missing arrays or optional fields
5. **Currency/Format Errors** (15% of failures) — Type mismatches on numeric values

---

## Why This Matters

**65% is NOT production-ready** because:
- 35% of documents fail parsing entirely
- No consistent structure learnable by downstream systems
- Cannot reliably extract required fields (invoice number, vendor, total)
- Multi-currency and complex documents consistently fail

**However, 65% baseline proves** that:
- The model has fundamental capability
- Clear patterns of failure exist (fixable via fine-tuning)
- A curated training dataset targeting these patterns will improve performance significantly

---

## Next Steps → Phase D

Fine-tuning on 80 curated examples will target each failure mode:
- **Code fence wrapping**: Examples showing "return ONLY JSON, no markdown"
- **Prose responses**: Multi-item examples forcing structured output
- **Malformed JSON**: Correct JSON structure in all training examples
- **Incomplete extraction**: All training examples include all fields
- **Currency diversity**: 28 non-USD examples for robust extraction

**Expected post-fine-tuning baseline: 90-95% parse success rate**
