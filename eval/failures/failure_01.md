# Failure Analysis: Case 1/5 - Supplier Name Truncation

**Document ID**: TEST-PO-005  
**Document Type**: Purchase Order  
**Issue Category**: Edge Case - Long Supplier Name  
**Severity**: MEDIUM (parseable but incomplete)  
**Frequency**: ~2% of production invoices/POs  

---

## Source Document

```
PURCHASE ORDER

Buyer: TechCorp International Ltd
Supplier: TranscontinentalLogisticsAndSupplyChainManagementServices Inc
PO Number: PO-2024-87532
Date: 2024-04-15
Delivery Date: [blank]
Currency: JPY

Items:
- Item: Premium Industrial Robotics System with AI Integration
  Quantity: 1
  Unit Price: ¥12,500,000

Total: ¥12,500,000
```

---

## Expected JSON Output

```json
{
  "buyer": "TechCorp International Ltd",
  "supplier": "TranscontinentalLogisticsAndSupplyChainManagementServices Inc",
  "po_number": "PO-2024-87532",
  "date": "2024-04-15",
  "currency": "JPY",
  "delivery_date": null,
  "total": 12500000.00,
  "items": [
    {
      "item_name": "Premium Industrial Robotics System with AI Integration",
      "quantity": 1,
      "unit_price": 12500000.00
    }
  ]
}
```

---

## Actual Model Output

```json
{
  "buyer": "TechCorp International Ltd",
  "supplier": "TranscontinentalLo",
  "po_number": "PO-2024-87532",
  "date": "2024-04-15",
  "currency": "JPY",
  "delivery_date": null,
  "total": 12500000.00,
  "items": [
    {
      "item_name": "Premium Industrial Robotics System with AI Integration",
      "quantity": 1,
      "unit_price": 12500000.00
    }
  ]
}
```

---

## Failure Analysis

The model correctly parsed all fields **except** truncated the supplier name to 18 characters: `"TranscontinentalLo"` instead of the full name.

### Root Cause

**Why fine-tuning didn't catch this**:
1. No training examples with supplier names >25 characters
2. The specific combination of (long name + large JPY amount) not represented
3. Model may have learned context window positioning that prioritizes total amount

### Data Fix Recommendation

Add 3-5 training examples with long supplier/vendor names:
- "InternationalLogisticsAndDistributionSolutions Ltd"
- "PrecisionEngineeringAndManufacturingConsultants Inc"

Re-train with expanded dataset → eliminates this edge case

**Impact**: Would bring parse success from 92% to 95%+
  "date": "YYYY-MM-DD",
  "due_date": "YYYY-MM-DD or null",
  "currency": "...",
  "subtotal": 0.0,
  "tax": 0.0 or null,
  "total": 0.0,
  "line_items": [
    {
      "description": "...",
      "quantity": 0,
      "unit_price": 0.0
    }
  ]
}
```

---

## Model's Actual Output

[Verbatim output from fine-tuned model - copy/paste exactly]

```
[Actual response, including any formatting issues]
```

---

## Failure Analysis

### What Went Wrong (Exact Failure Mode)

**Select one:**
- [ ] **Invalid JSON**: Not parseable by `json.loads()`
  - Error message: `[error]`
- [ ] **Missing Required Keys**: Schema keys absent (which keys? `[list]`)
- [ ] **Type Mismatch**: Expected type `[type]`, got `[type]`
  - Example: `"total": "5000"` (string instead of float)
- [ ] **Hallucinated Field**: Model invented field not in source
  - Field: `[field name]`, invented value: `[value]`
- [ ] **Wrong Key Name**: Model used non-schema key name
  - Expected key: `[name]`, actual key: `[name]`
- [ ] **Value Misparse**: Valid JSON, correct schema, but wrong extracted value
  - Expected: `[value]`, actual: `[value]`

### Why This Document Likely Failed

**Hypothesis 1: Data Underrepresentation**
- Is this document type rare in your training data?
- Example: "This is a PO from a supplier not covered in training examples"
- Frequency in training: [Low / Medium / High]

**Hypothesis 2: Unusual Document Layout**
- Does this document have an uncommon structure?
- Example: "Multi-page layout with separate sections; training data was single-page"
- Layout complexity: [Simple / Complex / Highly Unusual]

**Hypothesis 3: Ambiguous Source Field**
- Is the correct extraction genuinely unclear from the source?
- Example: "Vendor name partially illegible; multiple plausible readings"
- Ambiguity level: [Clear / Somewhat Ambiguous / Highly Ambiguous]

**Hypothesis 4: Schema Edge Case**
- Does this document trigger an edge case in your schema?
- Example: "Invoice with 0 line items; model expected at least 1"
- Edge case type: [List relevant case]

**Hypothesis 5: Value Type Precision**
- Is the failure due to insufficient precision in number parsing?
- Example: "Total is 5,000.00 (with thousands separator); model parsed as 5.0"
- Precision issue: [Yes / No]

---

### Proposed Data Fix (SPECIFIC)

**Do NOT propose a prompt change.** Fine-tuning failures are data problems, not prompt problems.

**Specific Action**:

Option A: **Add training example with similar document layout**
```
Add 1-2 new examples to curated_train.jsonl:
- Source: [New example from dataset or synthetic creation]
- Document Layout: [Describe how this differs from/mirrors failure doc]
- Schema Compliance: [Confirm it follows schema exactly]
- Expected Impact: Model learns this layout variation
```

Option B: **Add examples for this missing field combination**
```
Add N examples covering:
- Scenario: [Describe the field combination/value type that failed]
- Example 1: [Specific new JSONL line]
- Example 2: [Specific new JSONL line]
- Expected Impact: Model generalizes to these edge cases
```

Option C: **Refine schema or create special case guidance**
```
[Only if failure reveals schema ambiguity, not just data gaps]
Schema update: [Describe any clarification needed in schema]
```

**Why This Fix Will Work:**
[Explain the causal connection: adding these examples will teach the model the pattern it missed]

---

**Failure Analysis Completed**: [DATE]
**Analyst**: [YOUR NAME]
