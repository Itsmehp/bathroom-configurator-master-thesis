# Product Management API - Test Guide

This document contains test commands for the new Product Management API endpoints.

## Authentication

First, obtain a JWT token by logging in:

### PowerShell
```powershell
$body = @{email='admin@emc2.de'; password='admin123'} | ConvertTo-Json
$response = Invoke-RestMethod -Uri 'http://localhost:8000/api/auth/login' -Method Post -Body $body -ContentType 'application/json'
$token = $response.token
Write-Host "Token: $token"
```

### Curl (Bash/Linux)
```bash
TOKEN=$(curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@emc2.de","password":"admin123"}' \
  | jq -r '.token')
echo "Token: $TOKEN"
```

---

## API Endpoints

Base URL: `http://localhost:8000/api/product-management`

### 1. Create Product

**Endpoint:** `POST /api/product-management`

**Required Fields:**
- `name` (string)
- `modelNumber` (string) - must be unique
- `priceNet` (number)
- `width` (number)
- `productType` (string)

**Optional Fields:**
- `priceGross` (number) - defaults to priceNet * 1.19
- `depth` (number)
- `height` (number)
- `isBudget` (boolean) - defaults to false
- `productLink` (string)
- `openingType` (string) - GLEITUR, PANDELTUER, FLATPANELTUER, WALKIN_OHNE_TUER, DUSCHVORHANG
- `glassType` (string) - ESG_KLAR, ESG_NUBES, ESG_NEBULA
- `profileColor` (string) - SILBER_MATT, SILBER_HOCHGLANZ
- `style` (string) - NICHE, ECKEINSTIEG, U_FORM, WALKIN
- `orientation` (string) - LEFT, RIGHT, UNIVERSAL
- `isSidewall` (boolean) - defaults to false
- `minEinbau` (number)
- `maxEinbau` (number)
- `isShortSidewall` (boolean) - defaults to false

#### PowerShell
```powershell
$token = 'YOUR_JWT_TOKEN'
$body = @{
    name='Test Product'
    modelNumber='TEST001'
    priceNet=99.99
    width=800
    productType='Test Type'
    style='NICHE'
    isSidewall=$false
} | ConvertTo-Json

$response = Invoke-RestMethod -Uri 'http://localhost:8000/api/product-management' `
    -Method Post `
    -Body $body `
    -ContentType 'application/json' `
    -Headers @{Authorization="Bearer $token"}
$response | ConvertTo-Json -Depth 5
```

#### Curl
```bash
curl -X POST http://localhost:8000/api/product-management \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Product",
    "modelNumber": "TEST001",
    "priceNet": 99.99,
    "width": 800,
    "productType": "Test Type",
    "style": "NICHE",
    "isSidewall": false
  }'
```

---

### 2. Get Product by ID

**Endpoint:** `GET /api/product-management/:id`

#### PowerShell
```powershell
$token = 'YOUR_JWT_TOKEN'
$productId = 'PRODUCT_UUID'
$response = Invoke-RestMethod -Uri "http://localhost:8000/api/product-management/$productId" `
    -Method Get `
    -Headers @{Authorization="Bearer $token"}
$response | ConvertTo-Json -Depth 5
```

#### Curl
```bash
curl -X GET http://localhost:8000/api/product-management/PRODUCT_UUID \
  -H "Authorization: Bearer $TOKEN"
```

---

### 3. Update Product

**Endpoint:** `PUT /api/product-management/:id`

All fields are optional. Only provided fields will be updated. Default values (isBudget, isSidewall, isShortSidewall) remain unchanged if not provided.

#### PowerShell
```powershell
$token = 'YOUR_JWT_TOKEN'
$productId = 'PRODUCT_UUID'
$body = @{
    name='Updated Test Product'
    priceNet=149.99
    depth=900
    openingType='GLEITUR'
    glassType='ESG_KLAR'
} | ConvertTo-Json

$response = Invoke-RestMethod -Uri "http://localhost:8000/api/product-management/$productId" `
    -Method Put `
    -Body $body `
    -ContentType 'application/json' `
    -Headers @{Authorization="Bearer $token"}
$response | ConvertTo-Json -Depth 5
```

#### Curl
```bash
curl -X PUT http://localhost:8000/api/product-management/PRODUCT_UUID \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Test Product",
    "priceNet": 149.99,
    "depth": 900,
    "openingType": "GLEITUR",
    "glassType": "ESG_KLAR"
  }'
```

---

### 4. Delete Product

**Endpoint:** `DELETE /api/product-management/:id`

#### PowerShell
```powershell
$token = 'YOUR_JWT_TOKEN'
$productId = 'PRODUCT_UUID'
$response = Invoke-RestMethod -Uri "http://localhost:8000/api/product-management/$productId" `
    -Method Delete `
    -Headers @{Authorization="Bearer $token"}
$response | ConvertTo-Json
```

#### Curl
```bash
curl -X DELETE http://localhost:8000/api/product-management/PRODUCT_UUID \
  -H "Authorization: Bearer $TOKEN"
```

---

### 5. Upload CSV

**Endpoint:** `POST /api/product-management/csv/upload`

**CSV Format:**
```
name,modelNumber,priceNet,width,depth,height,productType,isBudget,productLink,openingType,glassType,profileColor,style,orientation,isSidewall,minEinbau,maxEinbau,isShortSidewall
```

**Required CSV Columns:**
- name
- modelNumber
- priceNet
- width
- productType

**Optional CSV Columns:**
- depth, height, isBudget, productLink, openingType, glassType, profileColor, style, orientation, isSidewall, minEinbau, maxEinbau, isShortSidewall

#### PowerShell
```powershell
$token = 'YOUR_JWT_TOKEN'
$csvData = @"
name,modelNumber,priceNet,width,productType,style,orientation,isSidewall
CSV Product 1,CSV001,199.99,1000,Duschkabine,ECKEINSTIEG,LEFT,false
CSV Product 2,CSV002,249.99,1200,Duschkabine,NICHE,UNIVERSAL,true
CSV Product 3,CSV003,299.99,900,Duschwanne,WALKIN,RIGHT,false
"@

$body = @{csvData=$csvData} | ConvertTo-Json
$response = Invoke-RestMethod -Uri 'http://localhost:8000/api/product-management/csv/upload' `
    -Method Post `
    -Body $body `
    -ContentType 'application/json' `
    -Headers @{Authorization="Bearer $token"}
$response | ConvertTo-Json -Depth 5
```

#### Curl
```bash
curl -X POST http://localhost:8000/api/product-management/csv/upload \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "csvData": "name,modelNumber,priceNet,width,productType,style,orientation,isSidewall\nCSV Product 1,CSV001,199.99,1000,Duschkabine,ECKEINSTIEG,LEFT,false\nCSV Product 2,CSV002,249.99,1200,Duschkabine,NICHE,UNIVERSAL,true"
  }'
```

---

## Test Results Summary

### ✅ Test 1: Create Product
- **Status:** PASSED
- **Result:** Product created with ID, all details properly saved
- **Default Values:** priceGross calculated (19% VAT), isSidewall=false applied

### ✅ Test 2: Update Product
- **Status:** PASSED
- **Result:** Product updated successfully, only specified fields changed
- **Behavior:** DoorDetail created when door fields provided

### ✅ Test 3: Get Product
- **Status:** PASSED
- **Result:** Product retrieved with all details including relationships

### ✅ Test 4: CSV Upload
- **Status:** PASSED
- **Result:** All 3 products created successfully from CSV
- **Summary:** total=3, successful=3, failed=0

### ✅ Test 5: Delete Product
- **Status:** PASSED
- **Result:** Product deleted successfully

### ✅ Test 6: Verify Deletion
- **Status:** PASSED
- **Result:** 404 Not Found error returned as expected

---

## Architecture

The implementation follows a clean, modular architecture:

```
backend/src/
├── routes/
│   └── productManagement.ts         # HTTP routes with Swagger docs
├── controllers/
│   └── productManagementController.ts # Request/response handling
├── services/
│   └── productManagementService.ts   # Business logic
├── middleware/
│   └── productValidation.ts         # Input validation
└── utils/
    └── csvParser.ts                 # CSV parsing utility
```

### Key Features:
- ✅ Proper separation of concerns (routes/controllers/services)
- ✅ Transaction-based operations for data consistency
- ✅ Comprehensive validation middleware
- ✅ CSV upload with batch processing and error handling
- ✅ Automatic priceGross calculation (19% VAT)
- ✅ Default value handling (isBudget, isSidewall, isShortSidewall)
- ✅ Automatic ProductType creation if not exists
- ✅ Full Swagger/OpenAPI documentation
- ✅ Proper error handling and HTTP status codes

---

## Swagger Documentation

Access the interactive API documentation at:
http://localhost:8000/api-docs

Look for the "Product Management" tag to see all endpoints with request/response schemas.
