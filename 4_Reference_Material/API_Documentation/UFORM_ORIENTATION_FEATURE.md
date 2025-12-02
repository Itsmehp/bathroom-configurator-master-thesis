# U-Form Orientation Constraint and Installation Type Feature

## Overview

Added orientation constraint selector and installation type filtering for U-Form configurations, specifically for Gleitür door type. When a user selects an orientation (LEFT, RIGHT, UNIVERSAL, or BOTH_SIDES), the system filters Tür 1 and Tür 2 products to show only those matching the selected orientation. Additionally, for Gleitür, products are automatically filtered by installation type:

- **Tür 1** (Wand zu Seitenwand) shows only `WALL_TO_PANEL` products
- **Tür 2** (Seitenwand zu Seitenwand) shows only `PANEL_TO_PANEL` products

## Changes Made

### 1. Database Schema (Already Exists)

The `UFormConfiguration` model in `schema.prisma` already has the `orientationConstraint` field:

```prisma
orientationConstraint OrientationConstraint @default(UNIVERSAL)

enum OrientationConstraint {
  LEFT
  RIGHT
  UNIVERSAL
  BOTH_SIDES
}
```

### 2. Frontend Components

#### New Component: `UFormOrientationSelector.tsx`

- **Location**: `frontend/src/app/[locale]/dashboard/configure-bathroom-ai/components/`
- **Purpose**: Provides a UI selector for orientation constraint
- **Features**:
  - Four orientation options with icons (Left, Right, Universal, Both Sides)
  - Visual feedback for selected option
  - Responsive grid layout (2 columns on mobile, 4 on desktop)

#### Updated Component: `UFormProductSections.tsx`

- **Added**: `orientationConstraint` prop

#### Updated Component: `UFormProductSections.tsx`

- **Added**: `orientationConstraint` prop
- **Added**: `filterDoorsByOrientationAndInstallation()` function
- **Logic**:
  - Only filters for GLEITUR door type
  - When orientation is not UNIVERSAL, filters doors to show only those with matching orientationConstraint
  - Products with UNIVERSAL or BOTH_SIDES orientation are always shown
  - **NEW**: For Gleitür, filters Tür 1 by `installationType = WALL_TO_PANEL`
  - **NEW**: For Gleitür, filters Tür 2 by `installationType = PANEL_TO_PANEL`
  - Applies both orientation and installation type filtering together

#### Updated Component: `ProductsDisplay.tsx`

- **Added**: State for `selectedUFormOrientation` (defaults to "UNIVERSAL")
- **Added**: Import and usage of `UFormOrientationSelector`
- **UI Changes**:
  - Orientation selector appears below the back button when Gleitür is selected
  - Selector is hidden for other door types
  - Selection resets to UNIVERSAL when user goes back to door type selection
- **Behavior**:
  - Re-fetches products when orientation changes
  - Clears product selections when orientation changes
  - Passes orientation to API call

### 3. Backend API

#### Updated Route: `/api/uform/u-form-products`

- **File**: `backend/src/routes/uform.ts`
- **Added Parameter**: `orientationConstraint` (optional)
- **Swagger Documentation**: Updated to include new parameter

#### Updated Controller: `UFormController.getUFormDimensionProducts()`

- **File**: `backend/src/controllers/uformController.ts`
- **Change**: Extracts `orientationConstraint` from query params
- **Passes**: Orientation to service layer

#### Updated Service: `UFormProductService.getUFormDimensionProducts()`

- **File**: `backend/src/services/uformProductService.ts`
- **Added Parameter**: `orientationConstraint?: string`
- **Logic**:
  - Only filters for GLEITUR door type
  - When orientation is specified (and not UNIVERSAL), filters doors
  - Shows products with matching orientation, UNIVERSAL, or BOTH_SIDES
  - **NEW**: For Gleitür, separates door candidates before dimension filtering:
    - `door1Candidates` filtered by `installationType = WALL_TO_PANEL`
    - `door2Candidates` filtered by `installationType = PANEL_TO_PANEL`
  - Filtering happens before dimension-based filtering

#### Updated API Client: `fetchUFormDimensionProducts()`

- **File**: `frontend/src/lib/api/products.ts`
- **Added Parameter**: `orientationConstraint?: string`

## User Flow

1. **Select U-Form Style**: User selects U_FORM cabin style
2. **Select Door Type**: User selects GLEITUR (orientation selector appears)
3. **Select Orientation**: User chooses LEFT, RIGHT, UNIVERSAL, or BOTH_SIDES
4. **View Filtered Products**:
   - **Tür 1** (Wand zu Seitenwand) products are filtered by:
     - Orientation constraint (if selected)
     - Installation type = `WALL_TO_PANEL` (automatic for Gleitür)
   - **Tür 2** (Seitenwand zu Seitenwand) products are filtered by:
     - Orientation constraint (if selected)
     - Installation type = `PANEL_TO_PANEL` (automatic for Gleitür)
   - Products with UNIVERSAL or BOTH_SIDES orientation are always visible
   - Seitenwand products remain unaffected by orientation
5. **Change Orientation**: Products refresh automatically when orientation changes

## Filtering Rules

### For GLEITUR Door Type:

**Orientation Filtering:**

- **LEFT selected**: Show doors with `orientationConstraint = LEFT | UNIVERSAL | BOTH_SIDES`
- **RIGHT selected**: Show doors with `orientationConstraint = RIGHT | UNIVERSAL | BOTH_SIDES`
- **UNIVERSAL selected**: Show all doors (no orientation filtering)
- **BOTH_SIDES selected**: Show doors with `orientationConstraint = BOTH_SIDES | UNIVERSAL`

**Installation Type Filtering (Automatic):**

- **Tür 1**: Only shows products with `installationType = WALL_TO_PANEL`
- **Tür 2**: Only shows products with `installationType = PANEL_TO_PANEL`

**Combined Filtering**: Both orientation AND installation type filters must match for a product to be shown.

### For Other Door Types:

- No orientation filtering applied
- Orientation selector is hidden

## Technical Details

### State Management

- Frontend maintains `selectedUFormOrientation` state
- Defaults to "UNIVERSAL" for backward compatibility
- Resets to "UNIVERSAL" when returning to door type selection

### API Integration

- Orientation constraint passed as query parameter
- Backend validates and applies filtering before dimension checks
- Response structure remains unchanged (door1[], door2[], seitenwand[])

### Performance Considerations

- Filtering happens server-side to reduce data transfer
- No additional database queries required
- Uses existing UFormConfiguration relationships

## Testing Recommendations

1. **Verify Orientation Selector Appears**:

   - Select U_FORM style
   - Select GLEITUR door type
   - Confirm orientation selector is visible

2. **Test Orientation Filtering**:

   - Select LEFT orientation
   - Verify only LEFT/UNIVERSAL/BOTH_SIDES doors appear in Tür 1 and Tür 2
   - Repeat for RIGHT orientation

3. **Test Installation Type Filtering**:

   - Select GLEITUR door type
   - Verify Tür 1 only shows WALL_TO_PANEL products
   - Verify Tür 2 only shows PANEL_TO_PANEL products
   - Check that products meet both orientation AND installation type criteria

4. **Test UNIVERSAL Mode**:

   - Select UNIVERSAL orientation
   - Verify all doors are shown (still filtered by installation type)

5. **Test Other Door Types**:

   - Select PANDELTUER or other types
   - Verify orientation selector is hidden
   - Verify products display normally

6. **Test State Reset**:
   - Select orientation
   - Click back button
   - Re-select GLEITUR
   - Verify orientation reset to UNIVERSAL

## Database Requirements

Ensure `UFormConfiguration` records have appropriate values:

**Orientation Constraint:**

- Products intended for left installation: `LEFT`
- Products intended for right installation: `RIGHT`
- Products that work on either side: `UNIVERSAL`
- Products that work on both sides simultaneously: `BOTH_SIDES`

**Installation Type (Critical for Gleitür):**

- Tür 1 (Wand zu Seitenwand): `WALL_TO_PANEL`
- Tür 2 (Seitenwand zu Seitenwand): `PANEL_TO_PANEL`
- Full wall-to-wall installation: `WALL_TO_WALL`
- Products that work on both sides simultaneously: `BOTH_SIDES`

## Future Enhancements

1. Add orientation icons/diagrams to help users understand the difference
2. Add tooltips explaining each orientation option
3. Consider adding orientation constraint to other door types if needed
4. Add visual preview of door orientation in product cards
