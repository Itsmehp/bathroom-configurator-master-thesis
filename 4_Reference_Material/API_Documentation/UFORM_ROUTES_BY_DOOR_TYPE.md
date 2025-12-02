# U-Form Routes by Door Type

## Overview

The U-Form Best Search API has been **reorganized into separate files** for each door type for better code management and maintainability. Each route is specific to one door type and automatically filters results accordingly.

## File Structure

```
backend/src/routes/
├── uformBest.ts                    # Main router (mounts all sub-routes)
├── uformGleitur.ts                 # Gleitür-specific route
├── uformPandeltuer.ts              # Pendeltür-specific route
└── uformPandeltuermitFestfeld.ts   # Pendeltür mit Festfeld-specific route
```

Each file is self-contained with:

- Its own route handler
- Swagger documentation
- Door type automatically set
- Complete U-Form search logic

## Available Routes

### 1. **Gleitür (Sliding Door)**

**Endpoint:** `POST /api/uform-best/gleitur/search`  
**File:** `src/routes/uformGleitur.ts`

**Description:** Search for U-Form configurations with Gleitür doors.

**Request Body:**

```json
{
  "width": 753,
  "depth": 750,
  "priceRange": {
    "min": 1000,
    "max": 3000
  },
  "orientationConstraint": "RIGHT" // Optional: LEFT, RIGHT, UNIVERSAL, BOTH_SIDES
}
```

**Note:** `doorType` is NOT required - route automatically uses `GLEITUR`.

---

### 2. **Pendeltür (Swing Door)**

**Endpoint:** `POST /api/uform-best/pandeltuer/search`
**File:** `src/routes/uformPandeltuer.ts`

**Description:** Search for U-Form configurations with Pendeltür doors.

**Request Body:**

```json
{
  "width": 1200,
  "depth": 900,
  "priceRange": {
    "min": 800,
    "max": 3000
  },
  "orientationConstraint": "LEFT" // Optional
}
```

**Note:** `doorType` is NOT required - route automatically uses `PANDELTUER`.

---

### 3. **Pendeltür mit Festfeld (Swing Door with Fixed Panel)**

**Endpoint:** `POST /api/uform-best/pandeltuer-mit-festfeld/search`  
**File:** `src/routes/uformPandeltuermitFestfeld.ts`

**Description:** Search for U-Form configurations with Pendeltür mit Festfeld doors.

**Request Body:**

```json
{
  "width": 1200,
  "depth": 900,
  "priceRange": {
    "min": 800,
    "max": 3000
  },
  "orientationConstraint": "LEFT" // Optional
}
```

**Note:** `doorType` is NOT required - route automatically uses `PANDELTUER_MIT_FESTFELD`.

---

### 4. **Flatpaneltür (Flat Panel Door)**

**Endpoint:** `POST /api/uform-best/flatpaneltuer/search`

**Description:** Search for U-Form configurations with Flatpaneltür doors.

**Request Body:**

```json
{
  "width": 1200,
  "depth": 900,
  "priceRange": {
    "min": 800,
    "max": 3000
  },
  "orientationConstraint": "LEFT" // Optional
}
```

**Note:** `doorType` is NOT required - route automatically uses `FLATPANELTUER`.

---

## Response Structure

All routes return the same response structure:

```json
{
  "tuer1": {
    /* Product object - WALL_TO_PANEL door */
  },
  "tuer2": {
    /* Product object - PANEL_TO_PANEL door */
  },
  "seitenwand": {
    /* Product object - side panel */
  },
  "duschwanne": {
    /* Product object or null */
  },
  "totalPriceNet": 2345.67,
  "widthReduction": 0,
  "depthReduction": 0,
  "tuer1MatchedViaExtendedRange": false,
  "tuer2MatchedViaExtendedRange": false,
  "seitenwandMatchedViaExtendedRange": false,
  "tuer1IsSondermass": false,
  "tuer2IsSondermass": false,
  "seitenwandIsSondermass": false,
  "searchCriteria": {
    "requestedWidth": 753,
    "requestedDepth": 750,
    "doorType": "GLEITUR", // Automatically set by route
    "orientationConstraint": "RIGHT",
    "priceRange": { "min": 1000, "max": 3000 },
    "foundTuer1Width": 750,
    "foundTuer2Width": 750,
    "foundSeitenwandWidth": 750,
    "foundDuschwanneWidth": 750,
    "foundDuschwanneDepth": 750,
    "widthDifferenceTuer2": 3,
    "depthDifferenceTuer1": 0,
    "depthDifferenceSeitenwand": 0,
    "tuer1MatchedViaExtendedRange": false,
    "tuer2MatchedViaExtendedRange": false,
    "seitenwandMatchedViaExtendedRange": false,
    "tuer1IsSondermass": false,
    "tuer2IsSondermass": false,
    "seitenwandIsSondermass": false,
    "logic": "..."
  }
}
```

## Key Features

### 1. **Separated Files for Better Code Management**

- Each door type has its own dedicated route file
- Easier to maintain and update individual door types
- Clear separation of concerns
- Reduced file size and complexity

### 2. **Door Type Automatic**

- Each route automatically sets the correct `doorType`
- No need to include `doorType` in request body
- Prevents user error from sending wrong door type

### 3. **Sondermaß Priority System**

All routes use the **unified 4-tier priority system**:

- **Priority 1:** Standard products with strict range
- **Priority 2:** Standard products with extended range (-15mm/+5mm)
- **Priority 3:** Sondermaß products with strict range
- **Priority 4:** Sondermaß products with extended range

### 4. **Dimension Checking**

- **Tür 1 (WALL_TO_PANEL):** Matches against **depth** using UFormConfiguration ranges
- **Tür 2 (PANEL_TO_PANEL):** Matches against **width** using UFormConfiguration ranges
- **Seitenwand:** Matches against **depth** using showerCabinDetail ranges

### 5. **Bug Fix Included**

All routes include the fix for Tür 2 dimension checking (now correctly checks width instead of depth).

## Migration Guide

### Old API Call:

```javascript
POST /api/uform-best/search
{
  "width": 753,
  "depth": 750,
  "doorType": "GLEITUR",  // ❌ Required in old API
  "priceRange": { ... },
  "orientationConstraint": "RIGHT"
}
```

### New API Call:

```javascript
POST /api/uform-best/gleitur/search
{
  "width": 753,
  "depth": 750,
  // doorType removed - automatic ✅
  "priceRange": { ... },
  "orientationConstraint": "RIGHT"
}
```

## Benefits

✅ **Better Code Organization:** Each door type in its own file  
✅ **Easier Maintenance:** Update one door type without affecting others  
✅ **Type Safety:** Route ensures only correct door type is searched  
✅ **Cleaner API:** No need to specify door type in request body  
✅ **Extensible:** Easy to add more door types (just create new file)  
✅ **Clear Separation:** Each door type has dedicated endpoint and file  
✅ **Reduced Complexity:** Smaller, focused files instead of one large file  
✅ **Swagger Documentation:** Each route has specific documentation

## Testing Examples

### Test Gleitür:

```bash
curl -X POST http://localhost:5000/api/uform-best/gleitur/search \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "width": 753,
    "depth": 750,
    "priceRange": {"min": 1000, "max": 3000},
    "orientationConstraint": "RIGHT"
  }'
```

### Test Pendeltür:

```bash
curl -X POST http://localhost:5000/api/uform-best/pandeltuer/search \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "width": 1200,
    "depth": 900,
    "priceRange": {"min": 800, "max": 3000}
  }'
```

## Error Responses

### 404 - No Configuration Found

```json
{
  "error": "No suitable U-Form Gleitür configuration found within the price range"
}
```

### 404 - Product Type Not Found

```json
{
  "error": "DUSCHKABINE product type not found"
}
```

### 500 - Server Error

```json
{
  "error": "Failed to fetch U-Form Gleitür configuration"
}
```

## Implementation Details

**Main Router:** `backend/src/routes/uformBest.ts`

- Imports and mounts all sub-routes
- Acts as a central hub for U-Form routes

**Individual Route Files:**

1. **`uformGleitur.ts`** - Gleitür search logic
2. **`uformPandeltuer.ts`** - Pendeltür search logic
3. **`uformPandeltuermitFestfeld.ts`** - Pendeltür mit Festfeld search logic

**Note:** Flatpaneltür route has been removed as requested.

Each route file:

1. Validates request body (width, depth, priceRange, orientationConstraint optional)
2. Sets `doorType` automatically based on the file
3. Searches for Tür 1 (WALL_TO_PANEL) matching depth
4. Finds compatible Tür 2 (PANEL_TO_PANEL) via ProductRelationship matching width
5. Finds compatible Seitenwand via ProductRelationship matching depth
6. Finds Duschwanne matching width and depth
7. Checks total price against range
8. Returns complete configuration with all flags

## Adding New Door Types

To add a new door type:

1. Create new file: `src/routes/uformNewDoorType.ts`
2. Copy structure from existing door type file
3. Update `doorType` constant to new type
4. Update Swagger documentation
5. Import and mount in `uformBest.ts`:
   ```typescript
   import uformNewDoorTypeRouter from "./uformNewDoorType";
   router.use("/new-door-type", uformNewDoorTypeRouter);
   ```

## Future Enhancements

- [ ] Add caching for frequently requested dimensions
- [ ] Add support for custom dimension ranges per door type
- [ ] Add batch search endpoint for multiple configurations
- [ ] Add sorting options (by price, quality, etc.)
