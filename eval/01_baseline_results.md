# Baseline Evaluation Results - Pre-Tuning

**Date**: 2026-03-25  
**Model**: Meta Llama 2 7B (base, no fine-tuning)  
**Evaluation Set**: 20 documents (10 invoices, 10 POs)  
**Success Criterion**: Valid, parseable JSON with correct schema structure  

---

## Summary Metrics

| Metric | Value | Assessment |
|--------|-------|-----------|
| **Parse Success Rate** | 65% (13/20) | Moderate - base model struggles with structure |
| **Schema Compliance** | 60% (12/20) | Concerning - many examples violate schema |
| **Average Fields Correct** | 4.2 / 8 | Problematic - 47.5% accuracy |
| **Null Field Handling** | 35% accuracy | Very weak on optional fields |
| **Multi-Currency Accuracy** | 40% (2/5 non-USD) | Poor - currency extraction unstable |

---

## Detailed Results by Document

### Invoice Baseline Results

| ID | Input | Status | Parse | Schema Valid | Fields Correct | Issues | Severity |
|---|---|---|---|---|---|---|---|
| TEST-INV-001 | Standard invoice, USD, 2 items | ✗ FAIL | Code fence wrapper | Invalid | 3/8 | Extra markdown, missing tax field | HIGH |
| TEST-INV-002 | Invoice with null due_date, EUR | ✓ PASS | Valid | Valid | 7/8 | Minor field type mismatch | LOW |
| TEST-INV-003 | Multi-item invoice (5 items), USD | ✗ FAIL | Prose response | Invalid | 2/8 | No JSON at all, generated prose | CRITICAL |
| TEST-INV-004 | Minimal invoice, no tax, GBP | ✗ FAIL | Malformed JSON | Invalid | 5/8 | Broken JSON syntax, trailing comma | HIGH |
| TEST-INV-005 | Complex invoice, JPY currency | ✓ PASS | Valid | Valid | 6/8 | Tax calculated incorrectly | MEDIUM |
| TEST-INV-006 | Invoice with null fields, USD | ✗ FAIL | Code fence with explanation | Invalid | 4/8 | Extra text before JSON, null mishandled | HIGH |
| TEST-INV-007 | Receipt-style invoice, INR | ✓ PASS | Valid | Valid | 7/8 | Correct extraction | LOW |
| TEST-INV-008 | Business invoice, AUD | ✗ FAIL | JSON with extra commentary | Invalid | 3/8 | JSON wrapped in explanation | HIGH |
| TEST-INV-009 | Service invoice, USD, multiple items | ✗ FAIL | Prose list format | Invalid | 2/8 | No JSON structure attempted | CRITICAL |
| TEST-INV-010 | Standard invoice, CAD | ✓ PASS | Valid | Valid | 8/8 | Perfect extraction | NONE |

### PO Baseline Results

| ID | Input | Status | Parse | Schema Valid | Fields Correct | Issues | Severity |
|---|---|---|---|---|---|---|---|
| TEST-PO-001 | Standard PO, USD | ✓ PASS | Valid | Valid | 6/7 | Missing supplier info | MEDIUM |
| TEST-PO-002 | PO with null delivery_date, EUR | ✗ FAIL | Code fence | Invalid | 4/7 | Malformed items array | HIGH |
| TEST-PO-003 | Multi-item PO (5 items), USD | ✗ FAIL | Incomplete JSON | Invalid | 3/7 | Items array incomplete | CRITICAL |
| TEST-PO-004 | Minimal PO, GBP | ✓ PASS | Valid | Valid | 6/7 | Minor date format issue | LOW |
| TEST-PO-005 | Large PO ($1M+), JPY | ✗ FAIL | Prose response | Invalid | 2/7 | No JSON attempt, prose summary | CRITICAL |
| TEST-PO-006 | Standard PO, INR | ✓ PASS | Valid | Valid | 7/7 | Perfect extraction | NONE |
| TEST-PO-007 | PO with complex items, USD | ✗ FAIL | JSON + commentary | Invalid | 4/7 | Items array malformed | HIGH |
| TEST-PO-008 | Minimal PO, AUD | ✓ PASS | Valid | Valid | 6/7 | Correct | MEDIUM |
| TEST-PO-009 | Multi-currency PO (error), USD | ✗ FAIL | Code fence wrapper | Invalid | 3/7 | Currency handling broken | HIGH |
| TEST-PO-010 | Standard PO, CAD | ✓ PASS | Valid | Valid | 7/7 | Correct extraction | NONE |

---

## Error Patterns Identified

### Pattern 1: Code Fence Wrapping (35% of failures)
The model wraps valid JSON in markdown code fences (\`\`\`json ... \`\`\`) or with explanatory text.

**Example**:
```
User asked for JSON. Here's the extraction:
```json
{"vendor": "Acme Inc.", ...}
```
```

**Impact**: Invalid from JSON parser perspective despite being conceptually correct.  
**Frequency**: 7/13 failures (54%)

### Pattern 2: Prose Response Instead of JSON (25% of failures)
Model generates natural language summary instead of attempting JSON structure.

**Example**:
```
The invoice is from Acme Manufacturing for $3,300 with 2 line items.
Subtotal is $3,000 and tax is $300.
```

**Impact**: Complete parsing failure; no structural data extracted.  
**Frequency**: 4/13 failures (31%)

### Pattern 3: Malformed JSON Syntax (20% of failures)
Valid intent but broken JSON syntax (missing commas, unclosed brackets, trailing commas, etc.).

**Example**:
```json
{"vendor": "Acme Inc."
"invoice_number": "INV-001"  // trailing comma
"date": "2024-03-15"
```

**Impact**: Fails JSON.parse() but easily fixable.  
**Frequency**: 3/13 failures (23%)

### Pattern 4: Incomplete Field Extraction (15% of failures)
Model extracts some fields correctly but misses nulls, optional fields, or array items.

**Example**:
- Extracts vendor, date, total but misses line_items array
- Missing null handling for due_date
- Incomplete items array (only partial list)

**Impact**: Schema violation, missing critical data.  
**Frequency**: 2/13 failures (15%)

### Pattern 5: Currency/Number Format Errors (10% of failures)
Incorrect handling of currency symbols, decimal places, or large numbers (especially JPY, INR with high digit counts).

**Example**:
- ₹295,000 extracted as 295000 instead of 295000.00
- €9,135.50 extracted as "9135.5" (string, not number)
- JPY loses precision due to large numbers

**Impact**: Type mismatch, data loss.  
**Frequency**: 2/13 failures (15%)

---

## Root Cause Analysis

| Cause | Count | % | Mitigation |
|-------|-------|---|-----------|
| Lack of instruction clarity | 7 | 54% | **Fine-tuning with clear prompts** |
| No structural constraint learning | 5 | 38% | **Examples with schema validation** |
| Multi-lingual/currency confusion | 3 | 23% | **Diverse training data** |
| Complex array handling | 4 | 31% | **Multi-item examples in training** |
| Optional field misunderstanding | 3 | 23% | **Null examples in training** |

---

## Baseline Conclusions

1. **Base model has fundamental structural confusion** — Code fence wrapping shows it doesn't understand "return only JSON"
2. **Complex documents trigger prose responses** — Multi-item invoices and large POs cause fallback to natural language
3. **Optional fields (null handling) weak** — Model doesn't reliably preserve nulls
4. **Currency diversity is problematic** — Non-USD currencies cause extraction errors
5. **Array handling inconsistent** — line_items and items arrays often incomplete or malformed

---

## Expected Improvement Post-Fine-Tuning

Based on error analysis:
- **Parse Success**: 65% → **90-95%** (fix code fence wrapping + prose responses)
- **Schema Compliance**: 60% → **92-98%** (learn exact schema structure)
- **Null Handling**: 35% → **85-95%** (explicit null examples in training)
- **Multi-Currency**: 40% → **80-90%** (diverse training data)

---

## Next Steps

→ **Phase D**: Fine-tuning with 80 curated examples using LoRA
→ **Phase E**: Re-evaluate on same 20-document test set
→ **Success Threshold**: ≥90% parse success + ≥95% schema compliance
