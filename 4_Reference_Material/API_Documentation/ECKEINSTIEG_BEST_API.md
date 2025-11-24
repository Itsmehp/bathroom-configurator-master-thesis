# ECKEINSTIEG BEST API Documentation

## Overview

The `eckeinstiegBest` route provides an intelligent search API for finding the best ECKEINSTIEG or compatible NICHE door combinations with side panels and trays within specified price ranges. This route implements a sophisticated priority-based selection algorithm that considers multiple factors including door type, installation ranges, pricing, and compatibility.

## API Endpoint

```
POST /eckeinstieg-best/search
```

## Authentication

Requires authentication via `authMiddleware`.

## Request Body

```typescript
{
  width: number;           // Required: Desired width in mm (must be positive)
  depth: number;           // Required: Desired depth in mm for side panel (must be positive)
  priceRange: {            // Required: Price constraints for complete set
    min: number;           // Minimum total price (non-negative)
    max: number;           // Maximum total price (must be >= min)
  };
  orientation?: "LEFT" | "RIGHT" | "UNIVERSAL";  // Optional: Door orientation preference
  openingType?: "GLEITUR" | "PANDELTUER" | "FLATPANELTUER" | "WALKIN_OHNE_TUER" | "DUSCHVORHANG";  // Optional: Door opening type
  isShortSidewall?: boolean;  // Optional: Filter for short sidewall compatibility
}
```

### Validation

The route uses Zod-based validation middleware (`validateEckeinstiegBestSearch`) that enforces:

- **Required fields**: `width`, `depth`, `priceRange` (with both min/max)
- **Positive numbers**: `width` and `depth` must be > 0
- **Price range logic**: `max >= min`, both non-negative
- **Enum validation**: `orientation` and `openingType` must match Prisma enums
- **Type safety**: All fields validated for correct types

**Validation errors return 400 status** with detailed error messages.

## Response Format

### Success Response (200)

```typescript
{
  best: ProductWithDetails;           // Selected door product
  sidePanel: ProductWithDetails;      // Compatible side panel
  sidePanelWidth: number | null;      // Actual side panel width used
  tray: Product | null;               // Compatible tray (may be null)
  totalPriceNet: number;              // Total price of the combination
  widthReduction: number;             // Width reduction from requested (mm)
  strictWidthRangeMatch: boolean;     // Whether door fits exact range
  widthMissDirection: "BELOW_MIN" | "ABOVE_MAX" | null;
  widthMissByMm: number;
  widthRangeMessage: string | null;   // Human-readable range explanation
  extendedDoorRangeUsed: boolean;     // Whether extended range was used
  extendedSidePanelRangeUsed: boolean;
  selectionMethod: string | null;     // Selection algorithm used
  searchCriteria: {                   // Detailed search metadata
    requestedWidth: number;
    requestedDepth: number;
    trayDepthToleranceMm: number;
    fallbackPlusMm: number;
    fallbackMinusMm: number;
    priceRange: { min: number; max: number };
    orientation?: string;
    openingType?: string;
    foundWidth: number;
    foundSidePanelWidth: number | null;
    widthDifference: number;
    depthDifference: number | null;
    sidePanelMatchedViaFallback: boolean;
    doorMatchedViaExtendedRange: boolean;
    sidePanelMatchedViaExtendedRange: boolean;
    sidePanelMatchedViaSondermass: boolean;
    strictWidthRangeMatch: boolean;
    widthMissDirection: "BELOW_MIN" | "ABOVE_MAX" | null;
    widthMissByMm: number;
    selectionMethod: string | null;
    logic: string;  // Algorithm explanation
  }
}
```

### Error Responses

- **400 Bad Request**: Validation failed (detailed error messages)
- **404 Not Found**: No suitable combination found within constraints
- **500 Internal Server Error**: Server/database errors

## Business Logic Algorithm

### Selection Priority System

The algorithm follows a hierarchical approach with 2 main priority levels:

#### Priority 1: Standard Doors (Non-Sondermaß)
- Includes doors from strict, extended, and fallback ranges
- Doors with installation range ≤ 98mm
- Sorted by price descending (highest price first)
- Prefers exact matches but accepts extended ranges

#### Priority 2: Sondermaß Doors
- Doors with installation range > 98mm
- Only used when no standard doors work
- More expensive but flexible

### Width Iteration Strategy

1. **Start with requested width**
2. **Iterate downward in 50mm steps** until valid combination found
3. **Minimum width**: 750mm (configurable)

### Door Matching Logic

For each width, doors are found via multiple strategies:

1. **Strict Range**: `minEinbau ≤ width ≤ maxEinbau`
2. **Extended Range**: Asymmetric tolerance (-15mm below min, +5mm above max)
3. **Sondermaß**: Wide-range doors for special cases

### Side Panel Selection

Three fallback levels for side panels:

1. **Strict Match**: Exact depth range match (non-Sondermaß only)
2. **Extended Match**: Asymmetric tolerance (±5mm/15mm)
3. **Fallback Match**: Closest smaller width (legacy behavior)
4. **Sondermaß**: Wide-range panels as last resort

### Tray Selection

- **Width**: Matches door width (or requested width for Sondermaß doors)
- **Depth**: Uses separate tolerance (10mm) for flexibility
- **Priority**: Smallest suitable tray with lowest price

## Key Features

### Intelligent Price Optimization
- Finds combinations within specified price range
- Prefers higher-priced doors when multiple options exist
- Considers total cost (door + side panel + tray)

### Width Flexibility
- Automatically reduces width in 50mm steps if exact match unavailable
- Tracks width reduction and provides detailed feedback
- Supports extended ranges for better compatibility

### Compatibility Assurance
- Validates door-side panel relationships via database constraints
- Ensures all components work together
- Handles orientation and opening type preferences

### Sondermaß Handling
- Special logic for wide-range "custom-cut" products
- Used as fallback when standard products don't fit
- More expensive but provides solutions for edge cases

## Related Components

### Services
- `doorSelectionService.ts`: Door finding algorithms
- `sidePanelTrayService.ts`: Side panel and tray selection
- `productRelationshipService.ts`: Product compatibility data
- **`eckeinstiegBestService.ts`: Core business logic for combination finding**

### Utilities
- `eckeinstiegUtils.ts`: Sorting, range analysis, validation helpers
  - **`isSondermassDoor()`**: Determines if a door is Sondermaß (wide range > 98mm)
  - **`sortProductsByPriceDescending()`**: Sorts products by price (highest first)
- `eckeinstiegBestValidation.ts`: Request validation middleware

### Configuration
- `productSearch.ts`: Search parameters and tolerances

### Types
- `eckeinstiegSearch.ts`: TypeScript interfaces and enums

## Testing

Comprehensive test suite in `eckeinstiegBest.test.ts`:
- Validation middleware testing (10 test cases)
- Error response validation
- Business rule enforcement

## Recent Improvements

### ✅ Service Layer Refactoring (2025-11-10)
- Moved core business logic (`findBestCombinationAtWidth`) to dedicated service file
- Improved separation of concerns between HTTP handling and business logic
- Enhanced maintainability and testability

### ✅ Code Reuse Optimization (2025-11-10)
- Extracted `isSondermassDoor()` utility function for reusable Sondermaß detection
- Added `sortProductsByPriceDescending()` for consistent price-based sorting
- Eliminated code duplication across service files

### ✅ Validation Middleware (2025-11-10)
- Replaced inline validation with Zod-based middleware
- Proper 400 status codes for validation errors
- Detailed error messages for API clients
- Consistent with other routes in codebase

### ✅ Test Coverage
- Added comprehensive validation tests
- All edge cases covered
- CI/CD integration ready

## Usage Examples

### Basic Search
```json
{
  "width": 1200,
  "depth": 900,
  "priceRange": {
    "min": 500,
    "max": 2000
  }
}
```

### Advanced Search with Preferences
```json
{
  "width": 1200,
  "depth": 900,
  "priceRange": {
    "min": 800,
    "max": 1500
  },
  "orientation": "LEFT",
  "openingType": "GLEITUR",
  "isShortSidewall": false
}
```

## Performance Considerations

- **Caching**: Product relationship data cached for 5 minutes
- **Database Optimization**: Efficient queries with proper indexing
- **Early Termination**: Stops searching when valid combination found
- **Memory Management**: Uses Sets for deduplication

## Future Enhancements

- [ ] Add more sophisticated pricing algorithms
- [ ] Implement user preference learning
- [ ] Add bulk search capabilities
- [ ] Enhance error categorization
- [ ] Add performance metrics/monitoring

---

**Last Updated**: November 10, 2025
**Route File**: `backend/src/routes/eckeinstiegBest.ts`
**Service File**: `backend/src/services/eckeinstiegBestService.ts`
**Validation**: `backend/src/middleware/eckeinstiegBestValidation.ts`
**Tests**: `backend/src/__tests__/eckeinstiegBest.test.ts`</content>
<parameter name="filePath">d:\EMC2\Final Application\my-fullstack-app\ECKEINSTIEG_BEST_API.md