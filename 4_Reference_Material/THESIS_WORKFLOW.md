# Master Thesis - Shower Enclosure Selection Logic Workflow

## Overview
This document provides a structured workflow for documenting the shower enclosure selection algorithm for your master thesis. The application implements a rule-based recommendation system for 4 shower types with intelligent filtering and scoring.

---

## Part 1: System Architecture & Foundation

### 1.1 High-Level Architecture
- **Location**: `backend/src/`
- **Key Components**:
  - Controllers: `backend/src/controllers/`
  - Services: `backend/src/services/`
  - Types/Interfaces: `backend/src/types/`
  - Database Models: `backend/prisma/schema.prisma`

### 1.2 Shower Entry Types
Your thesis focuses on these 4 shower enclosure types:

1. **ECKEINSTIEG** (Corner Entry) - Diagonal corner installation
2. **NICHE** - Recessed wall installation
3. **WALKIN** - Open walk-in shower (no door)
4. **UFORM** - U-shaped configuration with sliding/hinged doors

### 1.3 Selection Logic Phases
Every shower type follows this pattern:
1. **Input Validation** - Accept user parameters (width, depth, price range, etc.)
2. **Strict Search** - Find exact matches within strict tolerance
3. **Extended Search** - Fallback search with relaxed tolerance
4. **Sondermaß Search** - Custom-made option (Sondermaß = customizable doors)
5. **Scoring & Ranking** - Prioritize by quality, price, compatibility
6. **Best Selection** - Return optimal combination

---

## Part 2: Detailed Workflow - Study Each Shower Type

### Phase 1: ECKEINSTIEG (Corner Entry) Selection

**Files to Study (in order):**

#### Step 1.1: API Endpoint & Request Structure
```
File: backend/src/routes/eckeinstieg.ts
- Understand the API endpoint
- Study the request payload structure
- Note: EckeinstiegSearchRequest includes width, depth, priceRange, orientation, openingType
```

#### Step 1.2: Type Definitions
```
File: backend/src/types/eckeinstiegSearch.ts
- EckeinstiegSearchRequest - Input parameters
- ProductWithDetails - Enhanced product type with relationships
- DoorSelectionResult - Result structure
- SidePanelResult - Side panel matching result
```

#### Step 1.3: Core Selection Algorithm
```
File: backend/src/services/eckeinstiegBestService.ts
- Main logic: findBestEckeinstiegCombination()
- Width iteration algorithm (decreasing width tolerance)
- Door finding at each width level
- Side panel matching logic
- Price filtering
```

#### Step 1.4: Door Finding Logic
```
File: backend/src/services/doorSelectionService.ts
- findStrictEckeinstiegDoors() - Exact match doors
- findExtendedEckeinstiegDoors() - Relaxed tolerance doors
- findSondermassEckeinstiegDoors() - Custom-made doors

Business Rules:
- Strict: width must be between minEinbau and maxEinbau
- Extended: width ± 10mm tolerance
- Sondermaß: minEinbau to maxEinbau range > 98mm
```

#### Step 1.5: Side Panel Selection
```
File: backend/src/services/sidepanelService.ts
- findBestSidepanel() - Finds compatible side panels
- scoreSidepanelBySidepanelQuality() - Quality ranking

Key Logic:
- Side panel width = requested width - door width
- Must fit within panel's minEinbau/maxEinbau constraints
- Scored by product quality metrics
```

#### Step 1.6: All Best Results (BestAll Endpoint)
```
File: backend/src/services/eckeinstiegBestAllService.ts
- findAllBestEckeinstiegCombinations()
- Returns multiple valid combinations ranked by score
- More comprehensive than single "best" selection
```

#### Step 1.7: Utility Functions
```
File: backend/src/utils/eckeinstiegUtils.ts
- sortProductsByDistanceFromWidth()
- sortProductsByRangeTightness()
- isSondermassDoor()
- buildEinbauRangeFilter()
```

---

### Phase 2: NICHE Selection

**Files to Study:**

```
File: backend/src/services/nicheSelectionService.ts
- Similar structure to Eckeinstieg
- Key difference: NICHE style doors only
- findBestNicheCombination()
- findStrictNicheDoors() in doorSelectionService.ts

Database Queries:
- Filter: showerCabinDetail.style = 'NICHE'
- Otherwise same Einbau range logic as Eckeinstieg
```

**Key Document:**
```
File: NICHE_ROUTE_COMPREHENSIVE_DOCUMENTATION.md
- Already exists in your project
- Detailed breakdown of NICHE selection logic
- Business rules and database constraints
```

---

### Phase 3: WALKIN (Walk-In Shower) Selection

**Files to Study (in order):**

#### Step 3.1: Basic Search
```
File: backend/src/services/walkinService.ts
- searchWalkinDoors() - Find available WALKIN style doors
- WALKIN comes from: DUSCHKABINE or SEITENWAND product types
- No door is required (openingType can include "WALKIN_OHNE_TUER")
```

#### Step 3.2: Best Selection Algorithm
```
File: backend/src/services/walkinBestService.ts
- findBestWalkinCombination()
- Finds: WALKIN door + shower tray combination
- Width iteration with fallback tolerance
- Tray selection logic (independent of door)

Key Functions:
- findStrictWalkinDoors() - Exact width match
- findExtendedWalkinDoors() - Relaxed tolerance
- findBestTraySizeBySidepanelQuality() - Tray selection
```

#### Step 3.3: Tray Selection
```
File: backend/src/services/sidePanelTrayService.ts
- findBestTraySizeBySidepanelQuality()
- Selection criteria: width ≤ door.width AND depth ≤ door.depth
- But can be either stricter or wider

Business Rule:
- Prefer tray ≥ 70cm width minimum
- Score by product quality
```

#### Step 3.4: All Best Results
```
File: backend/src/services/walkinBestAllService.ts
- findAllBestWalkinCombinations()
- Returns ranked list of WALKIN + tray combinations
```

---

### Phase 4: UFORM (U-Shape) Selection

**Files to Study:**

#### Step 4.1: UFORM Specific Models
```
File: backend/prisma/schema.prisma
- UFormConfiguration model - Stores U-form specific rules
- UFormSideWall model - Side panel relationships
- Study the relationships between products and U-form configs
```

#### Step 4.2: Configuration Management
```
File: backend/src/services/uformConfigurationService.ts
- getUFormConfigurations()
- getUFormSideWalls()
- Manages which products can be used for U-form
- Links products to their U-form-specific constraints
```

#### Step 4.3: Best Selection Algorithm
```
File: backend/src/services/uformBestService.ts
- findBestUFormCombination()
- U-form has TWO door types:
  - GLEITUR (sliding door)
  - PANDELTUER (swing door)
- Selection logic for each door type

Key Logic:
- Find door with correct doorType
- Must have U-form configuration for that doorType
- Apply minEinbau/maxEinbau constraints
- Width iteration with fallback
```

#### Step 4.4: Product Utilities
```
File: backend/src/services/uformProductService.ts
- findUFormDoors()
- getCompatibleUFormSidewalls()
- Product filtering specific to U-form constraints
```

#### Step 4.5: All Best Results
```
File: backend/src/services/uformBestAllService.ts
- findAllBestUFormCombinations()
```

---

## Part 3: Cross-Cutting Concerns

### Common Patterns Across All Types

#### A. Width Iteration Algorithm
All types use this pattern:
```
current_width = requested_width
while current_width >= MIN_WIDTH && no_combination_found:
    try strict search at current_width
    if found:
        return result with STRICT classification
    
    try extended search at current_width
    if found:
        return result with EXTENDED classification
    
    try sondermaß search at current_width
    if found:
        return result with SONDERMASS classification
    
    current_width -= TOLERANCE_STEP (usually 5-10mm)
```

**Location**: `backend/src/routes/*Best.ts` files

#### B. Product Scoring & Ranking
```
File: backend/src/services/sidepanelService.ts
- scoreSidepanelBySidepanelQuality()
- Quality metrics: range tightness, price, product quality
- Ranking algorithm for sorting alternatives
```

#### C. Database Query Patterns
```
File: backend/src/config/productSearch.ts
- PRODUCT_SEARCH_CONFIG - Global configuration
- STRICT_TOLERANCE_MM
- FALLBACK_PLUS_MM / FALLBACK_MINUS_MM
- MIN_WIDTH, MAX_WIDTH thresholds
```

---

## Part 4: Visual Documentation & Flowcharts

### Existing Diagrams in Project
```
backend/charts/
├── findBestDoor.mmd          - Decision tree for door selection
├── findDoor.mmd              - General door finding algorithm
├── scoreDoorBySidepanelQuality.mmd - Scoring logic
├── get-plan-single.mmd       - Plan retrieval flow
├── get-plan-all.mmd          - All plans retrieval
└── ...other flows

Study these BEFORE creating your own diagrams
```

---

## Part 5: Testing & Validation

### Unit Tests Location
```
backend/src/__tests__/
├── eckeinstieg.test.ts       - ECKEINSTIEG tests
├── niche.test.ts             - NICHE tests
├── walkin.test.ts            - WALKIN tests
├── duschwanneService.test.ts - Tray tests
├── sidepanels.test.ts        - Side panel tests
└── walkinBestAll.test.ts     - WALKIN all results
```

**Strategy**: 
1. Run tests to understand expected behavior
2. Study test cases as mini-documentation
3. Create test scenarios for thesis examples

### Running Tests
```bash
cd backend
npm run test                    # Run all tests
npm run test -- eckeinstieg    # Specific test file
npm run test -- --coverage     # Coverage report
```

---

## Part 6: Data Flow & API Examples

### API Endpoint Structure
```
POST /api/eckeinstieg          - Find all ECKEINSTIEG options
POST /api/eckeinstiegBest      - Single best ECKEINSTIEG
POST /api/eckeinstiegBestAll   - All ranked ECKEINSTIEG options

POST /api/niche                - Find all NICHE options
POST /api/nicheBest            - Single best NICHE
POST /api/nicheBestAll         - All ranked NICHE options

POST /api/walkin               - Find all WALKIN options
POST /api/walkinBest           - Single best WALKIN + tray
POST /api/walkinBestAll        - All ranked WALKIN options

POST /api/uform                - Find all U-FORM options
POST /api/uformBest            - Single best U-FORM
POST /api/uformBestAll         - All ranked U-FORM options
```

**Request Example:**
```json
{
  "width": 1200,
  "depth": 800,
  "priceRange": { "min": 500, "max": 5000 },
  "orientation": "LEFT",
  "openingType": "GLEITUR"
}
```

---

## Part 7: Master Thesis Writing Strategy

### Chapter Structure Recommendation

**Chapter 1: System Overview**
- Architecture diagram
- Database schema overview
- 4 shower types introduction
- Key algorithms overview

**Chapter 2: Common Selection Framework**
- Width iteration algorithm (core pattern)
- Einbau range constraints
- Tolerance levels (strict/extended/sondermaß)
- Scoring methodology

**Chapter 3: ECKEINSTIEG Selection (Most Complex)**
- Detailed algorithm walkthrough
- Door selection process
- Side panel matching
- Complete decision flow diagram

**Chapter 4: NICHE Selection**
- Similarities/differences from ECKEINSTIEG
- Simplified algorithm
- Use existing NICHE_ROUTE_COMPREHENSIVE_DOCUMENTATION.md

**Chapter 5: WALKIN Selection**
- Unique challenges (no door requirement)
- Door + tray selection
- Independence of components

**Chapter 6: UFORM Selection**
- Configuration model overview
- Door type-specific logic
- Multi-constraint handling

**Chapter 7: Comparative Analysis**
- Algorithm complexity comparison
- Performance characteristics
- Edge cases & fallback strategies

**Chapter 8: Implementation Details**
- Database queries
- Query optimization
- Caching strategies
- Error handling

---

## Part 8: Code Walkthrough Checklist

Use this checklist to ensure comprehensive understanding:

### For Each Shower Type:

- [ ] Understand request/response types
- [ ] Trace the main service function
- [ ] Identify all tolerance levels (strict/extended/sondermaß)
- [ ] Find width iteration logic
- [ ] Locate product filtering queries
- [ ] Understand scoring algorithm
- [ ] Study fallback behavior
- [ ] Review test cases
- [ ] Check error handling
- [ ] Verify price filtering
- [ ] Understand orientation/opening type handling
- [ ] Document database constraints

### Cross-Type Items:

- [ ] Configuration constants (PRODUCT_SEARCH_CONFIG)
- [ ] Utility functions (sorting, filtering)
- [ ] Common type definitions
- [ ] API route structure
- [ ] Controller layer
- [ ] Error handling patterns
- [ ] Logging/debugging approach

---

## Part 9: Documentation Tools & Resources

### Helpful Commands for Analysis

**Search codebase:**
```bash
# Find all references to a function
grep -r "findBestEckeinstiegCombination" backend/src/

# Find all product type references
grep -r "ECKEINSTIEG\|NICHE\|WALKIN\|UFORM" backend/src/

# List all service files
find backend/src/services -name "*.ts" | sort
```

**Generate type documentation:**
```bash
# View full schema
cat backend/prisma/schema.prisma | less
```

**API Testing:**
```bash
# Start backend (if not running)
cd backend && npm run dev

# Test API endpoint
curl -X POST http://localhost:8000/api/eckeinstiegBest \
  -H "Content-Type: application/json" \
  -d '{
    "width": 1200,
    "depth": 800,
    "priceRange": { "min": 500, "max": 5000 }
  }'
```

---

## Part 10: Recommended Study Order

1. **Week 1**: System foundation
   - Architecture & database schema
   - Types & interfaces
   - Configuration constants

2. **Week 2**: ECKEINSTIEG deep dive
   - Complete algorithm study
   - Decision tree walkthrough
   - Test case analysis

3. **Week 3**: Other shower types
   - NICHE (simpler variant)
   - WALKIN (unique component)
   - UFORM (configuration-based)

4. **Week 4**: Cross-cutting concerns
   - Scoring & ranking
   - Performance optimization
   - Error handling

5. **Week 5**: Documentation & thesis writing
   - Create diagrams
   - Write algorithms
   - Generate examples

---

## Quick Reference: Key Files by Topic

| Topic | Files |
|-------|-------|
| **ECKEINSTIEG Logic** | eckeinstiegBestService.ts, doorSelectionService.ts, sidepanelService.ts |
| **NICHE Logic** | nicheSelectionService.ts, doorSelectionService.ts |
| **WALKIN Logic** | walkinBestService.ts, walkinBestAllService.ts, sidePanelTrayService.ts |
| **UFORM Logic** | uformBestService.ts, uformConfigurationService.ts, uformProductService.ts |
| **Types** | backend/src/types/*.ts |
| **Utilities** | eckeinstiegUtils.ts, productSearch config |
| **Database** | backend/prisma/schema.prisma |
| **Tests** | backend/src/__tests__/*.test.ts |
| **Routes/APIs** | backend/src/routes/*.ts |
| **Diagrams** | backend/charts/*.mmd |

---

## Notes for Thesis Writing

- **Emphasis**: The width iteration algorithm is the core innovation
- **Key Challenge**: Handling tolerance levels gracefully
- **Complexity**: ECKEINSTIEG is most complex (two-component: door + side panel)
- **Scalability**: System designed to extend to other bathroom fixtures
- **Performance**: Database indexing on Einbau ranges critical
- **User Experience**: Progressive fallback ensures results even with strict constraints

---

**Last Updated**: November 2025
**Repository**: bathroom-quote-gen
**Target Audience**: Master Thesis Research
