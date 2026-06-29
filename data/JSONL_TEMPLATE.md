# Sample JSONL Training Data Format

Each line in `curated_train.jsonl` must be a complete, valid JSON object with three fields: `instruction`, `input`, and `output`.

## Invoice Examples

```jsonl
{"instruction": "Extract all invoice fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "INVOICE\nVendor: Acme Manufacturing Inc.\nInvoice #: INV-2024-001\nDate: March 15, 2024\nDue Date: April 15, 2024\n\nBill To:\nCustomer Name\nAddress\n\nLine Items:\n- Widget Assembly (100 units @ $25.00 each): $2,500.00\n- Consulting Services (1 @ $500.00): $500.00\nSubtotal: $3,000.00\nTax (10%): $300.00\nTotal Due: $3,300.00\n\nCurrency: USD", "output": "{\"vendor\": \"Acme Manufacturing Inc.\", \"invoice_number\": \"INV-2024-001\", \"date\": \"2024-03-15\", \"due_date\": \"2024-04-15\", \"currency\": \"USD\", \"subtotal\": 3000.00, \"tax\": 300.00, \"total\": 3300.00, \"line_items\": [{\"description\": \"Widget Assembly\", \"quantity\": 100, \"unit_price\": 25.00}, {\"description\": \"Consulting Services\", \"quantity\": 1, \"unit_price\": 500.00}]}"}
{"instruction": "Extract all invoice fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "FACTURA\nProveedor: Tecnología Global S.L.\nNúmero: FAC-2024-515\nFecha: 12 de marzo de 2024\nFecha Vencimiento: 9 de abril de 2024\n\nConcepto: Licencia de Software (anual): €2,500.00\nIVA (21%): €525.00\nTotal: €3,025.00\n\nMoneda: EUR", "output": "{\"vendor\": \"Tecnología Global S.L.\", \"invoice_number\": \"FAC-2024-515\", \"date\": \"2024-03-12\", \"due_date\": \"2024-04-09\", \"currency\": \"EUR\", \"subtotal\": 2500.00, \"tax\": 525.00, \"total\": 3025.00, \"line_items\": [{\"description\": \"Licencia de Software (anual)\", \"quantity\": 1, \"unit_price\": 2500.00}]}"}
{"instruction": "Extract all invoice fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "INVOICE\nVendor: Office Supplies Plus\nInvoice Number: OS-2024-0847\nIssue Date: 2024-02-28\nDue Date: Not Specified\n\nItems:\n1. Copy Paper (10 reams): $125.00\n2. Printer Ink Cartridges (6 pack): $95.00\n3. Desk Organizers (5 units): $80.00\n4. Shipping: $25.00\n\nSubtotal: $325.00\nTax: Included in total\nTotal: $325.00\nCurrency: USD", "output": "{\"vendor\": \"Office Supplies Plus\", \"invoice_number\": \"OS-2024-0847\", \"date\": \"2024-02-28\", \"due_date\": null, \"currency\": \"USD\", \"subtotal\": 325.00, \"tax\": null, \"total\": 325.00, \"line_items\": [{\"description\": \"Copy Paper (10 reams)\", \"quantity\": 10, \"unit_price\": 12.50}, {\"description\": \"Printer Ink Cartridges (6 pack)\", \"quantity\": 6, \"unit_price\": 15.83}, {\"description\": \"Desk Organizers (5 units)\", \"quantity\": 5, \"unit_price\": 16.00}, {\"description\": \"Shipping\", \"quantity\": 1, \"unit_price\": 25.00}]}"}
```

## Purchase Order Examples

```jsonl
{"instruction": "Extract all purchase order fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "PURCHASE ORDER\nPO Number: PO-2024-5001\nBuyer: Global Tech Innovations Ltd.\nSupplier: Premium Industrial Supplies Inc.\nOrder Date: March 10, 2024\nRequested Delivery: April 10, 2024\n\nItems:\n1. Industrial Control Units (Model X-500) - Qty: 50 @ $500.00 each = $25,000.00\n2. Installation and Training - Qty: 1 @ $500.00 = $500.00\n\nTotal PO Value: $25,500.00\nCurrency: USD", "output": "{\"buyer\": \"Global Tech Innovations Ltd.\", \"supplier\": \"Premium Industrial Supplies Inc.\", \"po_number\": \"PO-2024-5001\", \"date\": \"2024-03-10\", \"delivery_date\": \"2024-04-10\", \"currency\": \"USD\", \"total\": 25500.00, \"items\": [{\"item_name\": \"Industrial Control Units (Model X-500)\", \"quantity\": 50, \"unit_price\": 500.00}, {\"item_name\": \"Installation and Training\", \"quantity\": 1, \"unit_price\": 500.00}]}"}
{"instruction": "Extract all purchase order fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "ORDEN DE COMPRA\nNúmero: OC-2024-082\nComprador: Construcciones Modernas S.A.\nProveedor: Acero y Materiales LTDA\nFecha: 20 de febrero de 2024\nEntrega Solicitada: 20 de marzo de 2024\n\nMateriales:\n- Acero Estructural (Vigas IPN 300) - 100 toneladas @ ₹50,000/ton = ₹5,000,000\n- Transporte: ₹250,000\n\nTotal: ₹5,250,000\nMoneda: INR", "output": "{\"buyer\": \"Construcciones Modernas S.A.\", \"supplier\": \"Acero y Materiales LTDA\", \"po_number\": \"OC-2024-082\", \"date\": \"2024-02-20\", \"delivery_date\": \"2024-03-20\", \"currency\": \"INR\", \"total\": 5250000.00, \"items\": [{\"item_name\": \"Acero Estructural (Vigas IPN 300)\", \"quantity\": 100, \"unit_price\": 50000.00}, {\"item_name\": \"Transporte\", \"quantity\": 1, \"unit_price\": 250000.00}]}"}
{"instruction": "Extract all purchase order fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.", "input": "PURCHASE ORDER\nPO #: ORD-2024-119\nFrom: Office Systems Corp\nTo: Tech Furniture Solutions\nDate: March 5, 2024\nNo delivery date specified\n\nOrdered Items:\n- Executive Conference Table (Walnut) x 2 @ £1,200 = £2,400\n- Office Chairs (Ergonomic, Black) x 12 @ £350 = £4,200\n- Monitor Stands (Dual, Adjustable) x 5 @ £275 = £1,375\n\nTotal: £7,975\nCurrency: GBP", "output": "{\"buyer\": \"Office Systems Corp\", \"supplier\": \"Tech Furniture Solutions\", \"po_number\": \"ORD-2024-119\", \"date\": \"2024-03-05\", \"delivery_date\": null, \"currency\": \"GBP\", \"total\": 7975.00, \"items\": [{\"item_name\": \"Executive Conference Table (Walnut)\", \"quantity\": 2, \"unit_price\": 1200.00}, {\"item_name\": \"Office Chairs (Ergonomic, Black)\", \"quantity\": 12, \"unit_price\": 350.00}, {\"item_name\": \"Monitor Stands (Dual, Adjustable)\", \"quantity\": 5, \"unit_price\": 275.00}]}"}
```

## Key Rules for JSONL Format

1. **One JSON object per line**: No newlines within the JSON object
2. **Instruction field**: Always the same extraction instruction (as shown above)
3. **Input field**: Raw, unstructured document text (can include line breaks, symbols, formatting)
4. **Output field**: Valid JSON object conforming exactly to the schema
5. **Valid JSON**: The output must be valid JSON that can be parsed by `json.loads()`
6. **Numbers as Numbers**: Use `25.00`, not `"25.00"` for floats; use `100`, not `"100"` for integers
7. **Dates in ISO Format**: Always `YYYY-MM-DD`
8. **Null Values**: Use `null` (not `"null"` or empty string) for absent optional fields
9. **No Trailing Commas**: JSON does not allow trailing commas

## Validation

Test your JSONL file with:

```bash
# Count lines
wc -l data/curated_train.jsonl

# Validate JSON on each line (Python)
python3 << 'EOF'
import json
with open('data/curated_train.jsonl', 'r') as f:
    for i, line in enumerate(f, 1):
        try:
            obj = json.loads(line)
            assert 'instruction' in obj and 'input' in obj and 'output' in obj
            # Verify output is valid JSON
            output_obj = json.loads(obj['output'])
        except Exception as e:
            print(f"Line {i} ERROR: {e}")
print("All lines are valid!")
EOF
```

---

**Template Created**: Use these patterns for all 80 training examples.
