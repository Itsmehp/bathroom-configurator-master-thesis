# Advanced Product Upload Script Guide

## Overview

The `uploadProductsAdvanced.ts` script allows you to upload products with all related details including Door Details and Shower Cabin Details. It handles both creating new products and updating existing ones.

## Features

- ✅ **Automatic Price Calculation**: Applies 5% inflation to priceNet and 19% VAT for priceGross
- ✅ **Update or Create**: Checks if product exists by modelNumber and updates/creates accordingly
- ✅ **Optional Related Tables**: Handles DoorDetail and ShowerCabinDetail automatically
- ✅ **Validation**: Validates all enum values and required fields
- ✅ **Detailed Logging**: Shows created/updated/failed status for each product
- ✅ **Error Handling**: Continues processing even if individual products fail

## CSV Format

### Required Columns

| Column          | Type   | Description                        | Example                   |
| --------------- | ------ | ---------------------------------- | ------------------------- |
| `name`          | string | Product name                       | "Premium Shower Door 120" |
| `modelNumber`   | string | Unique model identifier            | "SD120PRE"                |
| `priceNet`      | number | Base price (will be +5% inflation) | "450.00"                  |
| `width`         | number | Width in mm                        | "1200"                    |
| `productTypeId` | string | Must exist in ProductType table    | "door-type-id"            |

### Optional Columns

| Column        | Type    | Description           | Example             |
| ------------- | ------- | --------------------- | ------------------- |
| `depth`       | number  | Depth in mm           | "50"                |
| `height`      | number  | Height in mm          | "1900"              |
| `isBudget`    | boolean | Budget product flag   | "true"/"false"      |
| `productLink` | string  | Image filename or URL | "product-image.jpg" |

### Door Detail Columns (Optional)

| Column         | Type | Valid Values                                                       | Default     |
| -------------- | ---- | ------------------------------------------------------------------ | ----------- |
| `openingType`  | enum | GLEITUR, PANDELTUER, FLATPANELTUER, WALKIN_OHNE_TUER, DUSCHVORHANG | GLEITUR     |
| `glassType`    | enum | ESG_KLAR, ESG_NUBES, ESG_NEBULA                                    | ESG_KLAR    |
| `profileColor` | enum | SILBER_MATT, SILBER_HOCHGLANZ                                      | SILBER_MATT |

### Shower Cabin Detail Columns (Optional)

| Column        | Type    | Valid Values                       | Default                          |
| ------------- | ------- | ---------------------------------- | -------------------------------- |
| `style`       | enum    | NICHE, ECKEINSTIEG, U_FORM, WALKIN | - (required if any cabin detail) |
| `orientation` | enum    | LEFT, RIGHT, UNIVERSAL             | null                             |
| `isSidewall`  | boolean | true, false                        | false                            |

## Price Calculation

```
Input priceNet: 450.00
After 5% inflation: 450.00 * 1.05 = 472.50
Final priceGross: 472.50 * 1.19 = 562.28
```

## Usage

### 1. Prepare Your CSV File

Create a file named `products_advanced.csv` in the `backend/src/scripts/` directory.

### 2. Run the Script

```bash
# Navigate to backend directory
cd backend

# Run the script
npm run ts-node src/scripts/uploadProductsAdvanced.ts
# or
npx ts-node src/scripts/uploadProductsAdvanced.ts
# new script to only update price and add new products
npx ts-node src/scripts/updateProductsAdvanced.ts
```

### 3. Check Output

The script will show detailed progress:

```
📦 Found 5 products in CSV
🚀 Starting upload process...

✅ [1/5] CREATED: Premium Shower Door 120 (SD120PRE)
   📋 Door detail created: GLEITUR, ESG_KLAR, SILBER_MATT

🔄 [2/5] UPDATED: Corner Shower Cabin Left (CSC100L)
   📋 Door detail updated: PANDELTUER, ESG_NUBES, SILBER_HOCHGLANZ
   🚿 Shower cabin detail updated: ECKEINSTIEG, LEFT, Sidewall: false

❌ [3/5] FAILED: Invalid Product - ProductType invalid-id not found

🎉 Upload process complete!
📊 Summary:
   ✅ Created: 2
   🔄 Updated: 1
   ❌ Failed: 1
   📦 Total processed: 4
```

## Sample CSV Examples

### Basic Product (No Details)

```csv
name,modelNumber,priceNet,width,depth,height,productTypeId,isBudget,productLink,openingType,glassType,profileColor,style,orientation,isSidewall
Simple Shower Tray,ST90BASIC,120.00,900,900,,tray-type-id,true,tray-90.jpg,,,,,,
```

### Door with Door Details

```csv
name,modelNumber,priceNet,width,depth,height,productTypeId,isBudget,productLink,openingType,glassType,profileColor,style,orientation,isSidewall
Premium Glass Door,PGD120,450.00,1200,50,1900,door-type-id,false,door-120.jpg,PANDELTUER,ESG_NUBES,SILBER_HOCHGLANZ,,,
```

### Complete Shower Cabin

```csv
name,modelNumber,priceNet,width,depth,height,productTypeId,isBudget,productLink,openingType,glassType,profileColor,style,orientation,isSidewall
Corner Shower Cabin,CSC100,850.00,1000,1000,1900,cabin-type-id,false,cabin-100.jpg,GLEITUR,ESG_KLAR,SILBER_MATT,ECKEINSTIEG,LEFT,false
```

## Error Handling

- **Missing Required Fields**: Product skipped with error message
- **Invalid ProductType**: Product skipped, must exist in database
- **Invalid Enum Values**: Uses default values where possible
- **Database Errors**: Individual product failures don't stop the process

## Prerequisites

1. Database must be running and accessible
2. ProductType records must exist for all referenced IDs
3. Prisma client must be properly configured

## Notes

- Products are identified by `modelNumber` for updates
- All enum values are case-insensitive (converted to uppercase)
- Optional fields can be left empty in CSV
- Related details are only created if at least one relevant field is provided
