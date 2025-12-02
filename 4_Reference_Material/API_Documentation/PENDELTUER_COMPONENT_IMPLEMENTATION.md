# Pendeltür Recommendation Component Implementation

## Overview

Created a new React component `PendelturRecommendation` similar to `GleiturRecommendation` that displays recommended products for U-Form Pendeltür configurations using the `/api/uform-best/pandeltuer/search` endpoint.

## Files Created

### 1. PendelturRecommendation.tsx

**Location:** `frontend/src/app/[locale]/dashboard/configure-bathroom-ai/PendelturRecommendation.tsx`

**Features:**

- Displays 1 Pendeltür (door)
- Displays 2 identical Seitenwände (side panels)
- Displays 1 Duschwanne (shower tray)
- Shows Sondermaß badges when custom sizes are used
- Shows Extended Range badges when products match via extended ranges
- Displays total price
- Purple theme to distinguish from Gleitür (green theme)
- Error handling for no match scenarios
- Image display with fallback handling

## Files Modified

### 1. useUFormState.ts

**Location:** `frontend/src/app/[locale]/dashboard/configure-bathroom-ai/hooks/useUFormState.ts`

**Changes:**

- Added `pendelturRecommendation` state for storing Pendeltür API response
- Added `setPendelturRecommendation` setter function
- Added `pendelturRecommendationLoading` state for loading indicator
- Added `setPendelturRecommendationLoading` setter function
- All new states exported from the hook

### 2. RecommendationSections.tsx

**Location:** `frontend/src/app/[locale]/dashboard/configure-bathroom-ai/components/RecommendationSections.tsx`

**Changes:**

- Imported `PendelturRecommendation` component
- Added `showPendelturRecommendation` prop
- Added `pendelturRecommendationLoading` prop
- Added `pendelturRecommendation` prop
- Added `onPendelturRecommendationSelect` callback prop
- Added Pendeltür recommendation section that displays when `showPendelturRecommendation` is true
- Shows loading state while fetching
- Renders `PendelturRecommendation` component when data is available

### 3. ProductsDisplay.tsx

**Location:** `frontend/src/app/[locale]/dashboard/configure-bathroom-ai/ProductsDisplay.tsx`

**Changes:**

#### State Management

- Destructured `pendelturRecommendation`, `setPendelturRecommendation`, `pendelturRecommendationLoading`, `setPendelturRecommendationLoading` from `useUFormState` hook

#### API Fetching

- Updated `useEffect` to handle both Gleitür and Pendeltür recommendations
- Separate loading states for each door type
- Clears opposite recommendation when switching door types
- Calls `/api/uform-best/pandeltuer/search` endpoint for Pendeltür
- Passes all required parameters: width, depth, priceRange, orientationConstraint, doorType

#### Event Handlers

- Added `handlePendelturRecommendationSelect` function that:
  - Sets U-Form selections with 1 door and 2 side panels
  - Selects the Duschwanne if available
  - Calls parent callbacks (onDoorSelect, onSidePanelSelect, onDuschwanneSelect)
  - Saves selections to localStorage
  - Proceeds to next step via onNext callback

#### Component Usage

- Updated `RecommendationSections` component usage to include:
  - `showPendelturRecommendation` prop (true when U_FORM + PANDELTUER selected)
  - `pendelturRecommendationLoading` prop
  - `pendelturRecommendation` data prop
  - `onPendelturRecommendationSelect` callback

## API Integration

### Endpoint

```
POST /api/uform-best/pandeltuer/search
```

### Request Body

```typescript
{
  width: number,        // in mm
  depth: number,        // in mm
  priceRange: {
    min: number,
    max: number
  },
  orientationConstraint: "LEFT" | "RIGHT" | "UNIVERSAL" | "BOTH_SIDES",
  doorType: "PANDELTUER"
}
```

### Response Structure

```typescript
{
  tuer1: Product,                           // The Pendeltür
  seitenwand1: Product,                     // First side panel
  seitenwand2: Product,                     // Second identical side panel
  duschwanne: Product | null,               // Shower tray (optional)
  totalPriceNet: number,                    // Total price
  widthReduction: number,                   // Width adjustment made
  depthReduction: number,                   // Depth adjustment made
  tuer1MatchedViaExtendedRange?: boolean,   // Extended range flag for door
  seitenwandMatchedViaExtendedRange?: boolean, // Extended range flag for panels
  tuer1IsSondermass?: boolean,              // Custom size flag for door
  seitenwandIsSondermass?: boolean,         // Custom size flag for panels
  noMatchFound?: boolean,                   // Error flag
  message?: string                          // Error message
}
```

## User Flow

1. User selects "U-Form" cabin style
2. User selects "Pendeltür" as door type
3. Component automatically fetches recommendation from API based on:
   - Width and depth from measurements
   - Price range preferences
   - Selected orientation constraint
4. Recommendation card displays with purple theme showing:
   - 1 Pendeltür with image and specifications
   - 2 identical Seitenwände with images and specifications
   - 1 Duschwanne (if available)
   - Total price
   - Badges for Sondermaß or Extended Range if applicable
5. User clicks "Select Recommendation" button
6. All products are automatically selected and added to configuration
7. User proceeds to next step

## Visual Design

### Color Scheme

- **Card Background:** Purple-50 (`bg-purple-50`)
- **Border:** Purple-200 (`border-purple-200`)
- **Title Text:** Purple-800 (`text-purple-800`)
- **Description Text:** Purple-700 (`text-purple-700`)
- **Button:** Purple-600 with Purple-700 hover (`bg-purple-600 hover:bg-purple-700`)
- **Best Combination Badge:** Purple-100 background with Purple-800 text

### Product Cards

- White background with purple-200 borders
- 32px height images with object-contain
- Product name, article number, dimensions displayed
- Sondermaß badge: Amber theme
- Extended Range badge: Blue theme
- Price displayed at bottom

## Error Handling

### No Match Found

When API returns `noMatchFound: true`:

- Red-themed error card displayed
- Error title and description shown
- User prompted to adjust dimensions or price range

### Loading States

- Loading text displayed while fetching
- Separate loading states for Gleitür and Pendeltür prevent conflicts

## Testing Recommendations

1. **Happy Path:**

   - Select U-Form style
   - Select Pendeltür door type
   - Verify recommendation loads
   - Click "Select Recommendation"
   - Verify all products are selected
   - Verify navigation to next step

2. **No Match Scenario:**

   - Use dimensions that don't match any products (e.g., width: 998mm, depth: 786mm)
   - Verify error card displays
   - Verify 404 response handled gracefully

3. **Loading States:**

   - Monitor loading indicator appears/disappears correctly
   - Switch between Gleitür and Pendeltür rapidly
   - Verify only one loading at a time

4. **Sondermaß & Extended Range:**

   - Test with dimensions requiring custom sizes
   - Verify badges display correctly
   - Verify both standard and Sondermaß selections work

5. **Orientation Changes:**
   - Change orientation constraint
   - Verify recommendation updates accordingly

## Implementation Notes

- Component follows same pattern as GleiturRecommendation for consistency
- Uses existing translation hooks for internationalization
- Purple color scheme distinguishes Pendeltür from Gleitür (green) visually
- Integrates seamlessly with existing U-Form product selection flow
- localStorage persistence ensures selections survive page refreshes
- All TypeScript types properly defined and checked
