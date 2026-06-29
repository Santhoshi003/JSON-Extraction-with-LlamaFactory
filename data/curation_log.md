# Data Curation Log

This document records every decision made during data curation for the 80-example training dataset. It serves as an audit trail proving rigorous, manual review of each potential training example.

## Curation Standards

- **Schema Compliance**: Every kept example must conform exactly to the invoice or PO schema
- **Layout Diversity**: Examples should have varied document structures, column orders, and formatting
- **Optional Fields**: Include examples with missing tax, due_date, delivery_date (use `null`)
- **Multi-Item**: Include 10+ examples with 3+ line items / items array entries
- **Currency Diversity**: Include 5+ examples with non-USD currencies (GBP, EUR, INR, JPY)
- **Ground Truth**: Reject any example where the correct extraction is ambiguous or uncertain

## Curation Entries

| Example ID | Document Type | Source (Dataset + Record ID) | Decision | Schema Issues Found | Notes |
|------------|---------------|---------------------------|----------|-------------------|-------|
| INV-001 | Invoice | CORD v2 - receipt_001 | KEPT | None | Typical invoice, single currency (USD), complete fields |
| INV-002 | Invoice | CORD v2 - receipt_002 | KEPT | None | Missing due_date (set to null); good layout diversity |
| INV-003 | Invoice | SROIE - sroie_042 | REJECTED | date illegible, vendor name cut off | Ground truth too uncertain; would require guessing vendor |
| INV-004 | Invoice | Synthetic - inv_sample_15 | KEPT | None | Multi-item invoice (5 line items); EUR currency for diversity |
| PO-001 | PO | katanaml - po_synthetic_001 | KEPT | None | Complete PO with delivery date; USD |
| PO-002 | PO | katanaml - po_synthetic_002 | REJECTED | total does not match sum of items | Arithmetic inconsistency; rejected to ensure ground truth quality |
| ... | ... | ... | ... | ... | ... |

## Summary Statistics

- **Total Examples Reviewed**: [count]
- **Examples Kept**: 80 (target)
  - Invoices: 50
  - Purchase Orders: 30
- **Examples Rejected**: [count]
- **Rejection Rate**: [%]

## Diversity Audit

### Layout Diversity
- **Single-column layouts**: [count]
- **Multi-column layouts**: [count]
- **Table-formatted**: [count]
- **Different vendor formats**: [count]

### Optional Field Handling
- **Examples with null due_date**: ≥15 ✓ / ✗
- **Examples with null tax**: ≥10 ✓ / ✗
- **Examples with null delivery_date**: ≥10 ✓ / ✗

### Multi-Item Examples
- **Invoices with 3+ line items**: ≥10 ✓ / ✗
- **POs with 3+ items**: ≥10 ✓ / ✗

### Currency Diversity
- **USD**: [count]
- **GBP**: ≥1 ✓ / ✗
- **EUR**: ≥1 ✓ / ✗
- **INR**: ≥1 ✓ / ✗
- **JPY**: ≥1 ✓ / ✗
- **Other (specify)**: [count]

## Key Decisions & Rationale

### Decision 1: Handling Illegible Dates
**Rule**: If date is present but illegible, reject the example. If date can be reasonably inferred from context (e.g., "issued today 2024-03-15"), use ground truth date.
**Rationale**: Ground truth reliability is paramount; guessing vendor dates undermines data quality.

### Decision 2: Handling Missing Vendor/Supplier
**Rule**: If vendor/supplier is truly absent, use empty string `""`. If vendor is partially visible or can be inferred from document context, use the inferred vendor name.
**Rationale**: Empty string preserves schema compliance while marking absence.

### Decision 3: Multi-Currency Handling
**Rule**: Preserve original currency from document. If currency is ambiguous, default to USD with a note.
**Rationale**: Model must learn to handle multiple currencies; defaulting teaches overfitting to USD.

### Decision 4: Line Item / Items Array Edge Cases
**Rule**: If an invoice has a single line item (the entire invoice), include it as a 1-element array. Do not omit the array.
**Rationale**: Ensures model learns to always return an array, even if single-element.

## Notes for Implementer

- Use Python with JSON validation: `python3 -c "import json; json.loads(open('output.txt').read())"` to verify each example parses
- Track which dataset each example comes from for reproducibility
- If creating synthetic examples, mark source as "SYNTHETIC - [reason]"
- Aim for ~10% synthetic examples (8 total) if public datasets are insufficient

---

**Curation Completed**: [DATE]
**Curator**: [YOUR NAME]
**Total Effort**: [HOURS] hours
