# Invoice Schema Definition

This document specifies the JSON schema for all invoice training and evaluation examples. Every training example must conform exactly to this schema.

## Required Keys

| Key | Type | Format/Constraints | Handling of Absent Fields | Description |
|-----|------|-------------------|---------------------------|-------------|
| `vendor` | string | Free text, alphanumeric + common punctuation | Use empty string `""` if vendor name is illegible or absent | The name of the vendor/supplier issuing the invoice. Examples: "Acme Corp", "Tata Steel Ltd.", "Amazon Business" |
| `invoice_number` | string | Alphanumeric, may include special chars (-, /, #) | Use empty string `""` if no invoice number is present | The unique identifier for the invoice. Examples: "INV-2024-001", "2024/03/15-001", "PO#12345" |
| `date` | string | ISO 8601 format: `YYYY-MM-DD` | Cannot be absent; use current date if illegible (document ground truth decision) | The invoice issuance date. Always required. |
| `due_date` | string or null | ISO 8601 format: `YYYY-MM-DD` or `null` | Use `null` if no payment due date is specified in the document | The payment due date. May be absent in many invoices. |
| `currency` | string | 3-letter ISO 4217 code (uppercase): USD, GBP, EUR, INR, JPY, CAD, AUD, etc. | Use "USD" as default if currency is not specified | The currency in which invoice amounts are expressed. |
| `subtotal` | float | Decimal number (no currency symbol or comma separators) | Use `0.0` if subtotal is not explicitly stated but can be computed from line_items | Sum of all line item amounts before tax. Examples: `1500.00`, `125.5` |
| `tax` | float or null | Decimal number or `null` | Use `null` if tax is not itemized or absent in the document | The tax amount (e.g., GST, VAT, sales tax). May be absent. |
| `total` | float | Decimal number (no currency symbol or comma separators) | Cannot be absent; should be computed as subtotal + tax (if tax is present) | The final invoice amount due, including tax. Always required. |
| `line_items` | array of objects | Array (may be empty `[]`, but should typically contain 1+ items) | Use empty array `[]` if no itemized line items are present | Array of purchased items/services. Each object must have: `description` (string), `quantity` (number), `unit_price` (float) |

## Line Items Array Structure

Each object in the `line_items` array must have:

| Key | Type | Format | Handling of Absent Fields | Description |
|-----|------|--------|---------------------------|-------------|
| `description` | string | Free text | Use `""` if not present | Description of the good or service provided. Examples: "Software License (Annual)", "Office Supplies", "Consulting Services (Q1)" |
| `quantity` | number | Integer or float (e.g., `5`, `2.5`) | Use `1` if quantity is not specified | The quantity of the item. |
| `unit_price` | float | Decimal number | Use `0.0` if unit price cannot be determined | The price per unit. |

## Example Compliant Invoice JSON

```json
{
  "vendor": "Acme Manufacturing Ltd.",
  "invoice_number": "INV-2024-0342",
  "date": "2024-03-15",
  "due_date": "2024-04-15",
  "currency": "USD",
  "subtotal": 5000.00,
  "tax": 500.00,
  "total": 5500.00,
  "line_items": [
    {
      "description": "Widget Assembly",
      "quantity": 100,
      "unit_price": 25.00
    },
    {
      "description": "Shipping and Handling",
      "quantity": 1,
      "unit_price": 500.00
    }
  ]
}
```

## Curation Rules

1. **Consistency**: Every training example must follow this schema exactly. No missing keys (except `due_date` and `tax` which can be `null`), no extra keys.
2. **Data Diversity**: Include invoices with varying layouts, column orders, and field presence patterns to prevent the model from learning specific formatting rather than general extraction.
3. **Nullable Fields**: `due_date` and `tax` may be `null`, but all other fields must have a valid value.
4. **Number Format**: All floats must be actual JSON numbers, not strings. Never include currency symbols or thousands separators.
5. **Date Format**: All dates must be in `YYYY-MM-DD` format (ISO 8601).
6. **Required Presence**: `vendor`, `invoice_number`, `date`, `currency`, `subtotal`, `total`, and `line_items` must always be present and non-null.
