# U-Form Best Configuration API

## Overview

The `/api/uform-best/search` endpoint finds the optimal U-Form shower configuration based on customer requirements. Similar to `eckeinstieg-best`, this endpoint intelligently searches for the best combination of components that fit within specified dimensions and price range.

## Endpoint

**POST** `/api/uform-best/search`

**Authentication**: Required (Bearer token)

## Request Body

```json
{
  "width": 1200,
  "depth": 900,
  "priceRange": {
    "min": 800,
    "max": 3000
  },
  "doorType": "GLEITUR",
  "orientationConstraint": "LEFT"
}
```

### Parameters

| Parameter               | Type   | Required | Description                                                                 |
| ----------------------- | ------ | -------- | --------------------------------------------------------------------------- |
| `width`                 | number | Yes      | Desired width in mm (for Tür 2 - panel to panel door)                       |
| `depth`                 | number | Yes      | Desired depth in mm (for Tür 1 - wall to panel door and Seitenwand)         |
| `priceRange.min`        | number | Yes      | Minimum total price (net)                                                   |
| `priceRange.max`        | number | Yes      | Maximum total price (net)                                                   |
| `doorType`              | string | Yes      | One of: `GLEITUR`, `PANDELTUER`, `PANDELTUER_MIT_FESTFELD`, `FLATPANELTUER` |
| `orientationConstraint` | string | No       | One of: `LEFT`, `RIGHT`, `UNIVERSAL`, `BOTH_SIDES` (only for GLEITUR)       |

## Response Structure

```json
{
  "duschwanne": {
    "id": "uuid",
    "name": "Product Name",
    "width": 1200,
    "depth": 900,
    "priceNet": 350.00,
    ...
  },
  "tuer1": {
    "id": "uuid",
    "name": "Tür 1 - Wall to Panel",
    "width": 900,
    "priceNet": 580.00,
    "uFormConfigurations": [
      {
        "installationType": "WALL_TO_PANEL",
        "doorType": "GLEITUR",
        "orientationConstraint": "LEFT",
        "minEinbauMm": 850,
        "maxEinbauMm": 950
      }
    ],
    ...
  },
  "tuer2": {
    "id": "uuid",
    "name": "Tür 2 - Panel to Panel",
    "width": 1200,
    "priceNet": 620.00,
    "uFormConfigurations": [
      {
        "installationType": "PANEL_TO_PANEL",
        "doorType": "GLEITUR",
        "orientationConstraint": "LEFT",
        "minEinbauMm": 1150,
        "maxEinbauMm": 1250
      }
    ],
    ...
  },
  "seitenwand": {
    "id": "uuid",
    "name": "Seitenwand",
    "width": 900,
    "priceNet": 280.00,
    ...
  },
  "totalPriceNet": 1830.00,
  "widthReduction": 0,
  "depthReduction": 0,
  "tuer1MatchedViaExtendedRange": false,
  "tuer2MatchedViaExtendedRange": false,
  "seitenwandMatchedViaExtendedRange": false,
  "searchCriteria": {
    "requestedWidth": 1200,
    "requestedDepth": 900,
    "doorType": "GLEITUR",
    "orientationConstraint": "LEFT",
    "priceRange": { "min": 800, "max": 3000 },
    "foundTuer1Width": 900,
    "foundTuer2Width": 1200,
    "foundSeitenwandWidth": 900,
    "foundDuschwanneWidth": 1200,
    "foundDuschwanneDepth": 900,
    "widthDifferenceTuer2": 0,
    "depthDifferenceTuer1": 0,
    "depthDifferenceSeitenwand": 0,
    "tuer1MatchedViaExtendedRange": false,
    "tuer2MatchedViaExtendedRange": false,
    "seitenwandMatchedViaExtendedRange": false,
    "logic": "..."
  }
}
```

## How It Works

### 1. Component Roles

For **GLEITUR** (and other U-Form configurations):

- **Tür 1** (Wall to Panel):

  - Installation type: `WALL_TO_PANEL`
  - Matches the **depth** dimension
  - Connects the wall to the side panel

- **Tür 2** (Panel to Panel):

  - Installation type: `PANEL_TO_PANEL`
  - Matches the **width** dimension
  - Connects between two side panels

- **Seitenwand** (Side Panel):

  - Must have `COMPATIBLE_UFORM_SEITENWAND` relationship with Tür 1
  - Matches the **depth** dimension
  - Found through ProductRelationship table

- **Duschwanne** (Shower Tray):
  - Matches both width and depth with ±50mm tolerance
  - Optional component

### 2. Search Algorithm

```
1. Start with requested width and depth
2. Search for Tür 1 (WALL_TO_PANEL):
   - Filter by doorType
   - Filter by orientationConstraint (if GLEITUR)
   - Filter by installationType = WALL_TO_PANEL
   - Match against depth dimension

3. Find compatible Seitenwand:
   - Must be in ProductRelationship with Tür 1
   - RelationshipType = COMPATIBLE_UFORM_SEITENWAND
   - Match against depth dimension

4. Search for Tür 2 (PANEL_TO_PANEL):
   - Filter by doorType
   - Filter by orientationConstraint (if GLEITUR)
   - Filter by installationType = PANEL_TO_PANEL
   - Match against width dimension

5. Find Duschwanne:
   - Match width ± 50mm
   - Match depth ± 50mm

6. Calculate total price
7. If price within range: SUCCESS
8. If not: Reduce dimensions by 50mm and retry
```

### 3. Dimension Matching

**Strict Range** (Preferred):

- Product's `minEinbauMm` ≤ requested dimension ≤ `maxEinbauMm`

**Extended Range** (Fallback):

- `(minEinbauMm - 15mm)` ≤ requested dimension ≤ `(maxEinbauMm + 5mm)`
- Allows slightly outside the strict range
- Marked in response flags

**Dimension Reduction**:

- If no match found, reduce by 50mm
- Width minimum: 700mm
- Depth minimum: 700mm

### 4. Orientation Filtering (GLEITUR only)

When `orientationConstraint` is specified:

- Filters products by their `uFormConfigurations.orientationConstraint`
- Shows products matching: selected orientation OR UNIVERSAL OR BOTH_SIDES
- Example: If LEFT selected, shows LEFT, UNIVERSAL, and BOTH_SIDES products

## Database Requirements

### UFormConfiguration Table

Products must have proper U-Form configurations:

```sql
-- Example: Tür 1 (Wall to Panel) - LEFT orientation
INSERT INTO u_form_configurations (
  product_id,
  installation_type,
  door_type,
  orientation_constraint,
  required_doors,
  required_side_panels,
  min_einbau_mm,
  max_einbau_mm
) VALUES (
  'door-uuid',
  'WALL_TO_PANEL',  -- Critical for Tür 1
  'GLEITUR',
  'LEFT',
  2,
  1,
  850,
  950
);

-- Example: Tür 2 (Panel to Panel) - LEFT orientation
INSERT INTO u_form_configurations (
  product_id,
  installation_type,
  door_type,
  orientation_constraint,
  required_doors,
  required_side_panels,
  min_einbau_mm,
  max_einbau_mm
) VALUES (
  'door-uuid',
  'PANEL_TO_PANEL',  -- Critical for Tür 2
  'GLEITUR',
  'LEFT',
  2,
  1,
  1150,
  1250
);
```

### ProductRelationship Table

Seitenwand must be linked to Tür 1:

```sql
INSERT INTO product_relationships (
  source_product_id,  -- Tür 1 ID
  target_product_id,  -- Seitenwand ID
  relationship_type
) VALUES (
  'tuer1-uuid',
  'seitenwand-uuid',
  'COMPATIBLE_UFORM_SEITENWAND'
);
```

## Error Responses

### 400 Bad Request

```json
{
  "error": "Width must be a positive number"
}
```

### 404 Not Found

```json
{
  "error": "No suitable U-Form configuration found within price range and dimensions."
}
```

### 500 Internal Server Error

```json
{
  "error": "Failed to fetch best U-Form configuration"
}
```

## Example Usage

### Request

```bash
curl -X POST http://localhost:8000/api/uform-best/search \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "width": 1200,
    "depth": 900,
    "priceRange": {
      "min": 800,
      "max": 3000
    },
    "doorType": "GLEITUR",
    "orientationConstraint": "LEFT"
  }'
```

### Success Response

```json
{
  "duschwanne": { ... },
  "tuer1": { ... },
  "tuer2": { ... },
  "seitenwand": { ... },
  "totalPriceNet": 1830.00,
  "widthReduction": 0,
  "depthReduction": 0,
  "tuer1MatchedViaExtendedRange": false,
  "tuer2MatchedViaExtendedRange": false,
  "seitenwandMatchedViaExtendedRange": false,
  "searchCriteria": { ... }
}
```

## Key Differences from Eckeinstieg-Best

| Aspect             | Eckeinstieg-Best         | U-Form-Best                             |
| ------------------ | ------------------------ | --------------------------------------- |
| Components         | Door + Sidepanel + Tray  | Tür 1 + Tür 2 + Seitenwand + Duschwanne |
| Door Types         | 1 door                   | 2 doors (different installation types)  |
| Sidepanel          | Found by dimension match | Found via ProductRelationship           |
| Installation Types | N/A                      | WALL_TO_PANEL + PANEL_TO_PANEL          |
| Orientation        | Simple LEFT/RIGHT        | LEFT/RIGHT/UNIVERSAL/BOTH_SIDES         |
| Door Roles         | Single door              | Tür 1 (depth) + Tür 2 (width)           |

## Testing Checklist

- [ ] Test with GLEITUR + LEFT orientation
- [ ] Test with GLEITUR + RIGHT orientation
- [ ] Test with GLEITUR + UNIVERSAL orientation
- [ ] Test with other door types (PANDELTUER, etc.)
- [ ] Test price range filtering
- [ ] Test dimension reduction (request 1300mm, find 1250mm, 1200mm, etc.)
- [ ] Test extended range matching
- [ ] Verify Seitenwand relationship validation
- [ ] Verify all 4 components returned
- [ ] Test missing components (no duschwanne scenario)

## Files Created

- `backend/src/routes/uformBest.ts` - Main route handler
- `backend/src/types/uformBestSearch.ts` - TypeScript type definitions
- `backend/src/utils/uformBestUtils.ts` - Validation and utility functions
- `backend/src/services/uformBestService.ts` - Core search logic
- `backend/src/server.ts` - Route registration (updated)

## Performance Considerations

- Uses indexed database queries
- Filters applied at database level
- Iterative search with early termination
- Maximum iterations limited by dimension minimums (700mm)
- ProductRelationship lookup optimized

## Future Enhancements

1. Add caching for frequently requested configurations
2. Support for multiple Seitenwand (for some U-Form types)
3. Consider door compatibility between Tür 1 and Tür 2
4. Add weight/importance scoring for better "best" selection
5. Support for custom tolerance ranges
6. Batch search for multiple configurations
