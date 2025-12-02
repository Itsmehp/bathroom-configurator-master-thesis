# Local Image Hosting Guide

## Overview

Your product images are now served from the `public/Products/` directory in your Next.js frontend application.

## Directory Structure

```
frontend/
  public/
    Products/          # Place your product images here
      example-product.jpg
      another-product.png
```

## How to Add Product Images

### 1. File Placement

- Place all product images in: `frontend/public/Products/`
- Supported formats: `.jpg`, `.jpeg`, `.png`, `.webp`, `.gif`

### 2. Database Setup

Update your product records with the image filename in the `productLink` field:

```sql
-- Example: Update a product with an image
UPDATE "Product"
SET "productLink" = 'duschwanne-mineralguss-weiss-Nuovvo.jpg'
WHERE "id" = 'your-product-id';
```

### 3. Image Naming Convention

Use descriptive names that match your products:

- `product-model-number.jpg`
- `duschwanne-mineralguss-weiss-Nuovvo.jpg`
- `door-glass-120x200.png`

## Image Requirements

### Recommended Specifications

- **Aspect Ratio**: 4:3 or 16:9 works best
- **Resolution**: 400x300px minimum, 800x600px recommended
- **File Size**: Keep under 500KB for fast loading
- **Format**: JPG for photos, PNG for graphics with transparency

### Image Optimization Tips

1. Compress images before uploading (use tools like TinyPNG)
2. Use consistent aspect ratios for uniform appearance
3. Ensure good lighting and clear product visibility

## Fallback Behavior

- If no image is provided, a placeholder icon will be displayed
- If image fails to load, the placeholder will automatically appear
- Component handles both absolute paths (`/Products/image.jpg`) and relative paths (`image.jpg`)

## Local Development Access

Your images will be accessible at:

- `http://localhost:3000/Products/your-image.jpg`
- The Next.js Image component automatically optimizes and serves these images

## Production Deployment

For production, consider:

1. **CDN**: Upload images to AWS S3, Cloudinary, or similar
2. **Image Optimization**: Use Next.js built-in optimization
3. **Database Update**: Update `productLink` fields with full URLs
