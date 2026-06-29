# Purchase Order Schema Definition

This document specifies the JSON schema for all purchase order training and evaluation examples. Every training example must conform exactly to this schema.

## Required Keys

| Key | Type | Format/Constraints | Handling of Absent Fields | Description |
|-----|------|-------------------|---------------------------|-------------|
| `buyer` | string | Free text, alphanumeric + common punctuation | Use empty string `""` if buyer is not specified | The name of the organization placing the purchase order. Examples: "ABC Corporation", "Tech Solutions Inc.", "Global Enterprises" |
| `supplier` | string | Free text, alphanumeric + common punctuation | Use empty string `""` if supplier is not identified | The name of the supplier/vendor from whom goods/services are being procured. |
| `po_number` | string | Alphanumeric, may include special chars (-, /, #) | Use empty string `""` if no PO number is present | The unique purchase order identifier. Examples: "PO-2024-5001", "ORD/2024/003", "PR#88234" |
| `date` | string | ISO 8601 format: `YYYY-MM-DD` | Cannot be absent; use current date if illegible (document ground truth decision) | The date the purchase order was created/issued. Always required. |
| `delivery_date` | string or null | ISO 8601 format: `YYYY-MM-DD` or `null` | Use `null` if no requested delivery date is specified | The requested or expected delivery date. May be absent in many POs. |
| `currency` | string | 3-letter ISO 4217 code (uppercase): USD, GBP, EUR, INR, JPY, CAD, AUD, etc. | Use "USD" as default if currency is not specified | The currency in which the PO amounts are expressed. |
| `total` | float | Decimal number (no currency symbol or comma separators) | Cannot be absent; should be computed from items if necessary | The total PO value including all items. Always required. |
| `items` | array of objects | Array (may be empty `[]`, but should typically contain 1+ items) | Use empty array `[]` if no itemized items are present | Array of items being ordered. Each object must have: `item_name` (string), `quantity` (number), `unit_price` (float) |

## Items Array Structure

Each object in the `items` array must have:

| Key | Type | Format | Handling of Absent Fields | Description |
|-----|------|--------|---------------------------|-------------|
| `item_name` | string | Free text | Use `""` if not present | Name/description of the item being ordered. Examples: "Office Chairs (Black, Ergonomic)", "Server Hardware - Dell PowerEdge R750", "Consulting Services (Implementation)" |
| `quantity` | number | Integer or float (e.g., `50`, `2.5`) | Use `1` if quantity is not specified | The quantity being ordered. |
| `unit_price` | float | Decimal number | Use `0.0` if unit price cannot be determined | The price per unit. |

## Example Compliant Purchase Order JSON

```json
{
  "buyer": "Global Tech Innovations Ltd.",
  "supplier": "Premium Industrial Supplies",
  "po_number": "PO-2024-8847",
  "date": "2024-03-10",
  "delivery_date": "2024-04-10",
  "currency": "USD",
  "total": 25500.00,
  "items": [
    {
      "item_name": "Industrial Control Units (Model X-500)",
      "quantity": 50,
      "unit_price": 500.00
    },
    {
      "item_name": "Installation and Training",
      "quantity": 1,
      "unit_price": 500.00
    }
  ]
}
```

## Curation Rules

1. **Consistency**: Every training example must follow this schema exactly. No missing keys (except `delivery_date` which can be `null`), no extra keys.
2. **Data Diversity**: Include purchase orders with different suppliers, buyer organizations, item quantities, and layout variations.
3. **Nullable Fields**: Only `delivery_date` may be `null`. All other fields must have a valid value.
4. **Number Format**: All numbers must be actual JSON numbers, not strings. Never include currency symbols or thousands separators in numeric fields.
5. **Date Format**: All dates must be in `YYYY-MM-DD` format (ISO 8601).
6. **Required Presence**: `buyer`, `supplier`, `po_number`, `date`, `currency`, `total`, and `items` must always be present and non-null.
7. **Total Computation**: The `total` should represent the sum of all item quantities × unit prices (or as stated in the document if different).
