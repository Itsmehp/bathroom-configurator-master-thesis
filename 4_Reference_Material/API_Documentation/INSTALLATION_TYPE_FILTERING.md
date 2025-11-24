# Installation Type Filtering for U-Form Gleitür - Implementation Summary

## What Was Added

Extended the U-Form filtering logic to include **automatic installation type filtering** for Gleitür door types:

### Filtering Behavior

For **GLEITUR** door type:

- **Tür 1** (Wand zu Seitenwand): Only shows products with `installationType = WALL_TO_PANEL`
- **Tür 2** (Seitenwand zu Seitenwand): Only shows products with `installationType = PANEL_TO_PANEL`

This filtering works **in combination** with orientation constraint filtering.

## Code Changes

### Frontend

#### 1. `UFormProductSections.tsx`

**Function Renamed**: `filterDoorsByOrientation()` → `filterDoorsByOrientationAndInstallation()`

**New Parameters**:

```typescript
filterDoorsByOrientationAndInstallation(
  doorProducts: Product[],
  requiredInstallationType?: string  // NEW
)
```

**Updated Logic**:

```typescript
// Check installation type (only for GLEITUR and when specified)
const installationMatch =
  doorType !== "GLEITUR" ||
  !requiredInstallationType ||
  configInstallationType === requiredInstallationType;

return orientationMatch && installationMatch;
```

**Usage**:

```typescript
// Tür 1 filtered for WALL_TO_PANEL
const door1Products = filterDoorsByOrientationAndInstallation(
  doors.filter((door) => door.role === "door1"),
  doorType === "GLEITUR" ? "WALL_TO_PANEL" : undefined
);

// Tür 2 filtered for PANEL_TO_PANEL
const door2Products = filterDoorsByOrientationAndInstallation(
  doors.filter((door) => door.role === "door2"),
  doorType === "GLEITUR" ? "PANEL_TO_PANEL" : undefined
);
```

### Backend

#### 2. `uformProductService.ts`

**Method**: `getUFormDimensionProducts()`

**Updated Logic**:

```typescript
// For GLEITUR, filter door1 by WALL_TO_PANEL and door2 by PANEL_TO_PANEL
let door1Candidates = doors;
let door2Candidates = doors;

if (doorType === "GLEITUR") {
  // Filter door1 for WALL_TO_PANEL installation type
  door1Candidates = doors.filter((product: any) => {
    const configs = product.uFormConfigurations || [];
    return configs.some((config: any) => {
      return config.installationType === "WALL_TO_PANEL";
    });
  });

  // Filter door2 for PANEL_TO_PANEL installation type
  door2Candidates = doors.filter((product: any) => {
    const configs = product.uFormConfigurations || [];
    return configs.some((config: any) => {
      return config.installationType === "PANEL_TO_PANEL";
    });
  });
}

// Apply dimension filtering AFTER installation type filtering
door1Candidates = UFormProductUtils.filterProductsByDimension(
  door1Candidates,
  depth
);

door2Candidates = UFormProductUtils.filterProductsByDimension(
  door2Candidates,
  width
);
```

## Filter Execution Order

For Gleitür products, filters are applied in this sequence:

1. **Product Type Filter**: Only doors (not sidewalls)
2. **Orientation Constraint Filter** (if selected): LEFT/RIGHT/UNIVERSAL/BOTH_SIDES
3. **Installation Type Filter**:
   - Tür 1 → WALL_TO_PANEL
   - Tür 2 → PANEL_TO_PANEL
4. **Dimension Filter**:
   - Tür 1 → matches depth
   - Tür 2 → matches width

## Example Scenarios

### Scenario 1: LEFT orientation, 1200mm x 900mm

**Tür 1** must have:

- ✅ `orientationConstraint = LEFT | UNIVERSAL | BOTH_SIDES`
- ✅ `installationType = WALL_TO_PANEL`
- ✅ Dimensions fit 900mm depth

**Tür 2** must have:

- ✅ `orientationConstraint = LEFT | UNIVERSAL | BOTH_SIDES`
- ✅ `installationType = PANEL_TO_PANEL`
- ✅ Dimensions fit 1200mm width

### Scenario 2: UNIVERSAL orientation, any dimensions

**Tür 1** must have:

- ✅ `orientationConstraint = ANY` (no orientation filtering)
- ✅ `installationType = WALL_TO_PANEL`
- ✅ Dimensions match

**Tür 2** must have:

- ✅ `orientationConstraint = ANY` (no orientation filtering)
- ✅ `installationType = PANEL_TO_PANEL`
- ✅ Dimensions match

## Database Configuration Requirements

For Gleitür products to appear correctly, ensure `UFormConfiguration` records have:

```sql
-- Example for a door that can be used as Tür 1 on the left side
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
  'product-uuid',
  'WALL_TO_PANEL',     -- Critical for Tür 1
  'GLEITUR',
  'LEFT',              -- Or UNIVERSAL/RIGHT/BOTH_SIDES
  2,
  1,
  800,
  1200
);

-- Example for a door that can be used as Tür 2 on the right side
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
  'product-uuid',
  'PANEL_TO_PANEL',    -- Critical for Tür 2
  'GLEITUR',
  'RIGHT',             -- Or UNIVERSAL/LEFT/BOTH_SIDES
  2,
  1,
  1000,
  1400
);
```

## Benefits

1. **Architectural Accuracy**: Products are filtered by their actual installation position
2. **Better User Experience**: Users only see relevant products for each door position
3. **Reduced Confusion**: No more inappropriate products appearing in wrong positions
4. **Automatic Filtering**: No additional UI needed - happens automatically for Gleitür

## Backwards Compatibility

- ✅ Other door types (PANDELTUER, PANDELTUER_MIT_FESTFELD, FLATPANELTUER) are unaffected
- ✅ UNIVERSAL orientation still works, but respects installation type
- ✅ Existing data without proper installationType values will simply not appear (fail-safe)

## Testing Checklist

- [ ] Tür 1 shows only WALL_TO_PANEL products
- [ ] Tür 2 shows only PANEL_TO_PANEL products
- [ ] Orientation filtering still works with installation type filtering
- [ ] Other door types are unaffected
- [ ] Products appear in correct positions based on dimensions
- [ ] No products appear in wrong positions

## Files Modified

### Frontend

- ✅ `UFormProductSections.tsx` - Updated filtering function

### Backend

- ✅ `uformProductService.ts` - Added installation type filtering logic

### Documentation

- ✅ `UFORM_ORIENTATION_FEATURE.md` - Updated with installation type details

## Compilation Status

- ✅ Backend TypeScript compilation: **Success**
- ✅ Frontend errors: **None**
- ✅ Ready for deployment
