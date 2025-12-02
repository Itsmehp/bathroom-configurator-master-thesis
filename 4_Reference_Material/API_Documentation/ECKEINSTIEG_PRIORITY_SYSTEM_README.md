# ECKEINSTIEG Door Selection Priority System

## Overview

The ECKEINSTIEG door selection system implements a sophisticated priority-based algorithm to find the optimal door + side panel + tray combination for corner entry shower cabins. The system prioritizes standard doors over custom-cut (Sondermaß) doors and uses multiple fallback strategies to ensure a valid combination is always found within the specified price range.

## Core Algorithm Flow

### Main Entry Point: `findBestCombinationAtWidth()`

The algorithm follows a two-phase priority approach:

1. **PHASE 1 - Standard Doors**: Evaluate all standard doors (installation range ≤ 98mm) and find the highest-priced valid combination
2. **PHASE 2 - Sondermaß Doors**: Only if no standard doors work, evaluate Sondermaß doors (installation range > 98mm) for the highest-priced valid combination
3. **Width Reduction**: Only reduce width if no valid combination found at current width

### Key Behavioral Changes

**Sondermaß Priority Logic**:
- **Standard doors first**: Always prefer doors with narrow installation ranges (≤ 98mm) to avoid custom cutting
- **Sondermaß as fallback**: Only use wide-range Sondermaß doors when standard doors cannot provide a valid combination
- **Price optimization**: Within each phase, select the highest-priced combination that fits the price range

## Detailed Priority Breakdown

### Two-Phase Selection Process

#### Phase 1: Standard Doors (Installation Range ≤ 98mm)

**Door Collection**:
- Filters out Sondermaß doors (where `maxEinbau - minEinbau > 98mm`)
- Includes doors from strict range and extended range categories
- Sorts by `priceNet` descending (highest price first)

**Combination Building**:
For each standard door (highest price first):
1. **Side Panel Matching**: Finds compatible side panels using existing logic
2. **Tray Selection**: Selects tray based on door width
3. **Price Calculation**: Computes total price (door + side panel + tray)
4. **Range Validation**: Checks if total price fits within `priceRange.min` to `priceRange.max`

**Selection**: Returns the highest-priced valid combination from standard doors

#### Phase 2: Sondermaß Doors (Installation Range > 98mm) - Fallback Only

**Trigger Condition**: Only executed if Phase 1 finds no valid combinations

**Door Collection**:
- Includes only Sondermaß doors (where `maxEinbau - minEinbau > 98mm`)
- Sorts by `priceNet` descending (highest price first)

**Combination Building**:
For each Sondermaß door (highest price first):
1. **Side Panel Matching**: Finds compatible side panels (may include Sondermaß side panels)
2. **Tray Selection**: Uses requested width for tray selection (Sondermaß flexibility)
3. **Price Calculation**: Computes total price (door + side panel + tray)
4. **Range Validation**: Checks if total price fits within price constraints

**Selection**: Returns the highest-priced valid combination from Sondermaß doors

#### Key Behaviors
- **Exhaustive Search**: Evaluates all possible combinations at each width before reducing
- **Price Optimization**: Selects the most expensive valid combination at each width level
- **Width Reduction**: Only occurs when no combination fits the price range at current width

#### Door Selection Logic
- **ECKEINSTIEG_EXTENDED**: ECKEINSTIEG doors with asymmetric extended range:
  - `minEinbau ≤ currentWidth + 5mm` (allows 5mm above max)
  - `maxEinbau ≥ currentWidth - 15mm` (allows 15mm below min)
- **NICHE_EXTENDED**: NICHE doors with same extended range logic
- **Random Selection**: Randomly chooses between available extended groups
- **Sorting**: Within selected group, sorts by width descending
- **Extended Range Flag**: Marked as `doorMatchedViaExtendedRange = true`

## Width Iteration Strategy

### Main Search Loop
```typescript
let currentWidth = width; // Start with requested width
while (currentWidth >= MIN_WIDTH && !foundCombo) {
  // Try to find combination at current width
  // If no door found, reduce by 50mm steps
  currentWidth -= 50;
}
```

### Key Behaviors
- **Width Reduction**: Decreases width in 50mm steps until a valid combination is found
- **Tried Door Tracking**: Prevents trying the same door multiple times if price exceeds max
- **Price Validation**: Only accepts combinations where `totalPrice ≤ priceRange.max && totalPrice ≥ priceRange.min`

## Selection Method Indicators

### Base Methods
- `ECKEINSTIEG_STRICT`: Standard ECKEINSTIEG door in strict range
- `NICHE_STRICT`: Standard NICHE door in strict range
- `ECKEINSTIEG_SONDERMASS`: Custom-cut ECKEINSTIEG door
- `NICHE_SONDERMASS`: Custom-cut NICHE door
- `ECKEINSTIEG_EXTENDED`: ECKEINSTIEG door in extended range
- `NICHE_EXTENDED`: NICHE door in extended range

### Modified Methods (Width Reduction)
When the final door doesn't fit the original requested width, methods are modified:
- `_STRICT` → `_WIDTH_REDUCED`
- `_EXTENDED` → `_WIDTH_REDUCED`

## Sorting and Ranking Logic

### Door Sorting (Comprehensive)
1. **All Doors Combined**: Collects from strict, Sondermaß, and extended ranges
2. **Price Descending**: Sorts by `priceNet` descending (highest price first)
3. **Combination Evaluation**: Tests each door for valid side panel/tray combinations
4. **Total Price Selection**: Chooses highest total combination price within range

### Side Panel Sorting
- Same range tightness logic as doors
- Prioritizes non-Sondermaß side panels

### Tray Sorting
- Smallest width that accommodates requirements
- Lowest price as secondary criteria

## Configuration Constants

```typescript
PRODUCT_SEARCH_CONFIG = {
  TRAY_DEPTH_TOLERANCE_MM: 10,    // Tray depth flexibility
  FALLBACK_PLUS_MM: 5,            // Extended range above max
  FALLBACK_MINUS_MM: 15,          // Extended range below min
  WIDTH_STEP_MM: 50,              // Width reduction steps
  MIN_WIDTH: 750,                 // Minimum search width
}
```

## Response Metadata

### Width Analysis
- `strictWidthRangeMatch`: Whether final door fits original requested width
- `widthMissDirection`: "BELOW_MIN" or "ABOVE_MAX" if not matching
- `widthMissByMm`: Amount of mismatch in millimeters
- `widthRangeMessage`: Human-readable explanation in German

### Matching Flags
- `doorMatchedViaExtendedRange`: Door used extended/Sondermaß logic
- `sidePanelMatchedViaExtendedRange`: Side panel used extended logic
- `sidePanelMatchedViaSondermass`: Side panel is Sondermaß type
- `sidePanelMatchedViaFallback`: Side panel used closest-smaller logic

## Database Relationships

### Product Types
- **DUSCHKABINE**: Door products (source products)
- **DUSCHWANNE**: Tray products

### Relationship Types
- `REQUIRES_SIDE_PANEL`: Door requires specific side panel
- `COMPATIBLE_DOOR`: Alternative side panel compatibility

## Performance Optimizations

### Caching
- Source product IDs cached for 5 minutes
- Product type information cached in memory

### Query Optimization
- Filters applied at database level where possible
- Relationship queries use indexed fields
- Distinct selections to avoid duplicates

## Error Handling

### Validation
- Required fields: width, depth, priceRange.min, priceRange.max
- Type checking for orientation and opening types

### Fallback Logic
- Graceful degradation through priority levels
- Comprehensive null checking
- Detailed error messages for debugging

## Business Logic Rationale

1. **Cost Optimization**: Prioritizes standard doors to minimize manufacturing costs (no custom cutting)
2. **Sondermaß as Last Resort**: Only uses expensive Sondermaß doors when standard options are unavailable
3. **Price Maximization**: Within each phase, selects the highest-priced combination that fits constraints
4. **Comprehensive Evaluation**: Tests all possible combinations at each width before reducing dimensions
5. **Width-First Approach**: Exhausts all options at requested dimensions before accepting compromises

This system ensures users get the best possible combination within their constraints while maintaining business profitability through intelligent prioritization of standard products over custom manufacturing.