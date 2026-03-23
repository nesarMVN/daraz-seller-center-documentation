# Category Schema Integration

## Where the Schema Lives

Category schemas are defined in `moveon-category-schemas.json`. Each entry is keyed by the category path string (e.g., `"Access Control System > Smart Card"`) and contains:

```json
{
  "variation_attributes": [
    {
      "name": "Card Type",
      "required": true,
      "options": ["RFID 125kHz", "RFID 13.56MHz", "..."]
    }
  ],
  "specification_attributes": [
    {
      "name": "Brand",
      "required": true,
      "type": "text",
      "options": []
    },
    {
      "name": "Frequency",
      "required": false,
      "type": "select",
      "options": ["125kHz", "13.56MHz", "Dual Frequency", "Other"]
    }
  ],
  "moveon": {
    "id": "9ce8d7f8-1400-4950-b247-750df997fda3",
    "name": "Smart Card",
    "path": ["Electronics & Gadgets", "Access Control System", "Smart Card"]
  }
}
```

## How the Backend Accesses the Schema

The `CategorySchemaValidator` service maintains a lookup map from `moveon.id` (the shipping category UUID) to the schema definition. At application boot or first access, it loads the JSON file and builds this map. When a product is created or updated, the validator:

1. Receives the `shipping_category_id` from the request
2. Looks up the schema by category UUID in the map
3. If no schema found for this category → skip schema validation (not all categories have schemas yet; the product can still be created with freeform properties and specs)

## Validation Rules Derived from Schema

**Property validation (when schema exists):**

| Rule                                        | Check                                                                                         |
| ------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Required axes present                       | Every `variation_attribute` with `required: true` must have a matching entry in `properties[]` |
| Property names in schema                    | Each submitted `properties[].name` must match a name in the schema's `variation_attributes`    |
| Values in allowed options                   | Each submitted value must be in the corresponding attribute's `options` list, or "Other"/"Custom" if the schema includes those as options |

**Specification validation (when schema exists):**

| Rule                                        | Check                                                                                         |
| ------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Required specs present                      | Every `specification_attribute` with `required: true` must have a matching entry in `specifications[]` |
| Select-type value in options                | For specs with `type: select`, the submitted value must be in the schema's `options` list      |
| Text-type value unconstrained               | For specs with `type: text`, any non-empty string is valid                                     |

**When no schema exists for a category:** All properties and specifications are accepted without schema-specific validation. Standard field validations (max length, required fields, unique names) still apply.

## Schema Validation Error Response

When schema validation fails, the response includes specific details about which fields violated which rules:

```json
{
  "message": "Category schema validation failed.",
  "errors": {
    "properties.0.name": ["The property 'Weight' is not defined in the category schema for 'Smart Card'. Allowed properties: Card Type, Color."],
    "specifications.0.value": ["The specification 'Frequency' requires one of: 125kHz, 13.56MHz, Dual Frequency, Other. Got: 'custom-value'."]
  }
}
```
