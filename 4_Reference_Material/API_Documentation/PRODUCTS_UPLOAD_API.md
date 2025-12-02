# Products Upload API

This document describes the new product upload functionality that allows administrators to upload new products via CSV files.

## Endpoints

### POST /api/products-upload

Uploads new products from a CSV file. Only creates new products - skips existing model numbers.

**Authentication:** Requires admin authentication
**Content-Type:** multipart/form-data
**Query Parameters:**
- `dryRun=true` (optional): Perform validation without actually creating products

**CSV Format:**
The CSV file should contain the following columns (some are optional):

**Required Fields:**
- `modelNumber` - Unique product model number
- `name` - Product name
- `priceNet` - Net price (numeric)
- `width` - Product width in mm (numeric)
- `productTypeId` - Product type ID

**Optional Fields:**
- `priceGross` - Gross price (numeric, defaults to priceNet if not provided)
- `depth` - Product depth in mm (numeric)
- `height` - Product height in mm (numeric)
- `productLink` - URL to product image/link
- `isBudget` - Boolean flag for budget products ('true'/'false')

**Shower Cabin Specific Fields:**
- `style` - Cabin style ('NICHE', 'ECKEINSTIEG', 'U_FORM', 'WALKIN')
- `orientation` - Orientation ('LEFT', 'RIGHT', 'UNIVERSAL')
- `minEinbau` - Minimum installation dimension (numeric)
- `maxEinbau` - Maximum installation dimension (numeric)
- `isSidewall` - Boolean flag for side panels ('true'/'false')
- `isShortSidewall` - Boolean flag for short sidewalls ('true'/'false')

**Shower Tray Specific Fields:**
- `cuttable` - Boolean flag for cuttable trays ('true'/'false')

**Door Specific Fields:**
- `openingType` - Door opening type ('GLEITUR', 'PANDELTUER', 'FLATPANELTUER', 'WALKIN_OHNE_TUER', 'DUSCHVORHANG')
- `glassType` - Glass type ('ESG_KLAR', 'ESG_NUBES', 'ESG_NEBULA')
- `profileColor` - Profile color ('SILBER_MATT', 'SILBER_HOCHGLANZ')

**Response:**
```json
{
  "logs": ["Processing logs..."],
  "summary": {
    "created": 5,
    "skipped": 2,
    "failed": 0,
    "total": 7,
    "createdProducts": ["MODEL001", "MODEL002", ...],
    "skippedProducts": ["EXISTING001", "EXISTING002"]
  }
}
```

### POST /api/products-fetch (Future Implementation)

This endpoint is prepared for future functionality to run Python Selenium scripts based on provided model numbers.

**Authentication:** Requires admin authentication
**Body:**
```json
{
  "modelNumbers": ["MODEL001", "MODEL002"],
  "dryRun": false
}
```

**Response:**
```json
{
  "message": "Python script execution not yet implemented",
  "modelNumbers": ["MODEL001", "MODEL002"],
  "dryRun": false,
  "status": "pending"
}
```

## Usage Examples

### Upload Products with Dry Run
```bash
curl -X POST "http://localhost:8000/api/products-upload?dryRun=true" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "file=@products.csv"
```

### Upload Products (Production)
```bash
curl -X POST "http://localhost:8000/api/products-upload" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "file=@products.csv"
```

## CSV Example

```csv
name,modelNumber,priceNet,width,depth,height,productTypeId,isBudget,productLink,openingType,glassType,profileColor,style,orientation,isSidewall,minEinbau,maxEinbau,isShortSidewall
Gleittür f.Seitenwand 5.0 re.1000x2000mm silber hochglanz ESG klar m.ACS VIGOUR,V5GT100RHL,862.84,1000,,2000,11536e92-5811-4dee-8422-1bd2b08995f1,false,V5GT.jpg,GLEITUR,ESG_KLAR,SILBER_HOCHGLANZ,ECKEINSTIEG,RIGHT,false,980,1005,false
```

## Error Handling

- **400 Bad Request:** Invalid CSV format, missing required fields, invalid enum values
- **403 Forbidden:** User is not an admin
- **413 Payload Too Large:** File exceeds 10MB limit
- **500 Internal Server Error:** Database or processing errors

## Validation Rules

- Model numbers must be unique (existing products are skipped)
- Required numeric fields must be positive numbers
- Enum fields must match predefined values
- File size limited to 10MB
- Only CSV files accepted

## Future Enhancements

- Integration with Python Selenium scripts for automated data fetching
- Frontend interface for model number input and script execution
- Batch processing improvements
- Enhanced validation and error reporting