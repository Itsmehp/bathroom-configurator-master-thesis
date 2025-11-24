# Product Management Frontend Implementation Guide

## Files to Create

### 1. Main Page Component
**Path:** `frontend/src/app/[locale]/dashboard/product-management/page.tsx`

```typescript
import { Metadata } from "next";
import ProductManagementContent from "./ProductManagementContent";

export const metadata: Metadata = {
  title: "Product Management | EMC2",
  description: "Manage products with advanced search, filtering, and CSV import",
};

export default function ProductManagementPage() {
  return <ProductManagementContent />;
}
```

### 2. Product Filters Component
**Path:** `frontend/src/app/[locale]/dashboard/product-management/components/ProductFilters.tsx`

### 3. Product Form Dialog (Add/Edit)
**Path:** `frontend/src/app/[locale]/dashboard/product-management/components/ProductFormDialog.tsx`

### 4. CSV Upload Dialog
**Path:** `frontend/src/app/[locale]/dashboard/product-management/components/CSVUploadDialog.tsx`

## Features Implemented

✅ **Pagination**: 20 products per page (configurable: 10, 20, 50, 100)
✅ **Debounced Server Search**: 500ms delay, searches across all products
✅ **Instant Local Search**: Filters current page by name/model number
✅ **Advanced Filters**: Product Type, Cabin Style, Orientation
✅ **Add/Edit Products**: Full form with all fields
✅ **Delete Products**: With confirmation dialog
✅ **CSV Upload**: Bulk import with success/failure reporting
✅ **Responsive Design**: Mobile-friendly table and filters
✅ **Loading States**: Spinners and disabled states
✅ **Error Handling**: Alert dialogs for user feedback

## Navigation

Add to your dashboard navigation/sidebar:

```typescript
{
  title: "Product Management",
  href: "/dashboard/product-management",
  icon: Package, // from lucide-react
}
```

## API Integration

The frontend uses these API endpoints (already created):
- `GET /api/product-management/:id` - Get single product
- `POST /api/product-management` - Create product
- `PUT /api/product-management/:id` - Update product
- `DELETE /api/product-management/:id` - Delete product
- `POST /api/product-management/csv/upload` - CSV import
- `GET /api/products/paginated` - Get paginated products

## Testing

1. Start the backend: `cd backend && npm run dev`
2. Start the frontend: `cd frontend && npm run dev`
3. Navigate to `/dashboard/product-management`
4. Test all features:
   - Search products
   - Filter by type/style/orientation
   - Add new product
   - Edit existing product
   - Delete product
   - Upload CSV file

## CSV Format

```csv
name,modelNumber,priceNet,width,productType,style,orientation,isSidewall
Product 1,M001,199.99,1000,Duschkabine,NICHE,LEFT,false
Product 2,M002,299.99,1200,Duschwanne,ECKEINSTIEG,RIGHT,true
```

Required columns: name, modelNumber, priceNet, width, productType
Optional columns: all others
