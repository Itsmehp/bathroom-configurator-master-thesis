# Products Fetch API - Seamless Integration Guide

## Overview

This document explains the refactored architecture for seamlessly fetching and creating products from model numbers provided by the frontend.

## Architecture Flow

```
┌─────────────┐
│  Frontend   │
│             │
│ Sends:      │
│ {           │
│   modelNum  │
│   bers: []  │
│   dryRun?   │
│ }           │
└──────┬──────┘
       │
       │ POST /api/products-fetch
       ▼
┌─────────────────────────────────────┐
│  Backend (productsUpload.ts)        │
│                                     │
│  1. Validates modelNumbers array    │
│  2. Spawns Python CLI script        │
│  3. Passes model numbers as args    │
│  4. Captures stdout/stderr          │
│  5. Reads generated CSV             │
│  6. Validates & creates products    │
│  7. Returns summary + logs          │
└──────┬──────────────────────────────┘
       │
       │ spawn python enrich_models_cli.py MODEL1 MODEL2 ...
       ▼
┌─────────────────────────────────────┐
│  Python (enrich_models_cli.py)      │
│                                     │
│  1. Accepts CLI arguments           │
│  2. Starts Selenium WebDriver       │
│  3. Fetches each model:             │
│     - Search & capture API          │
│     - Extract prices                │
│     - Extract einbau from popup     │
│  4. Builds enriched CSV             │
│  5. Writes enriched_products_*.csv  │
│  6. Prints "OUTPUT_CSV:filename"    │
│  7. Exit code 0 on success          │
└─────────────────────────────────────┘
```

## Key Components

### 1. Frontend Integration

**Request Format:**
```typescript
POST /api/products-fetch
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "modelNumbers": ["V5WG100LHL", "V5GT110RHL", "V5GTS2LHL"],
  "dryRun": false  // optional, default false
}
```

**Response Format:**
```typescript
{
  "message": "Python script executed and products created successfully",
  "summary": {
    "created": 3,
    "skipped": 0,
    "failed": 0,
    "total": 3,
    "createdProducts": ["V5WG100LHL", "V5GT110RHL", "V5GTS2LHL"],
    "skippedProducts": [],
    "pythonScript": "path/to/enrich_models_cli.py",
    "generatedCsv": "enriched_products_20251103_150524.csv"
  },
  "logs": [
    "Starting Python script execution for 3 model numbers",
    "Model numbers: V5WG100LHL, V5GT110RHL, V5GTS2LHL",
    ...
  ]
}
```

### 2. Backend (`productsUpload.ts`)

**New `/products-fetch` Endpoint:**

- **Input**: `{ modelNumbers: string[], dryRun?: boolean }`
- **Process**:
  1. Validates user is admin
  2. Validates modelNumbers array
  3. Spawns Python CLI script with model numbers as arguments
  4. Captures Python output (stdout/stderr)
  5. Parses `OUTPUT_CSV:filename` from stdout
  6. Reads generated CSV file
  7. Validates and creates products in database
  8. Returns summary + detailed logs

**Key Features:**
- ✅ No CSV file required from frontend
- ✅ Model numbers passed directly as CLI args
- ✅ Automatic CSV generation by Python
- ✅ Dry run support (validation only)
- ✅ Comprehensive logging
- ✅ Transaction-based product creation

### 3. Python CLI Script (`enrich_models_cli.py`)

**Usage:**
```bash
python enrich_models_cli.py MODEL1 MODEL2 MODEL3 ...
```

**Features:**
- ✅ Accepts model numbers as CLI arguments (no CSV needed)
- ✅ Uses same scraping logic as `enrich_models.py`
- ✅ Generates timestamped CSV: `enriched_products_YYYYMMDD_HHMMSS.csv`
- ✅ Prints CSV filename to stdout: `OUTPUT_CSV:filename.csv`
- ✅ Exit code 0 on success, 1 on failure
- ✅ Comprehensive logging to stdout

**Workflow:**
1. Parse CLI arguments
2. Start Chrome WebDriver
3. Login to website
4. For each model:
   - Search and capture API response
   - Extract prices (net/gross)
   - Extract einbau values from popup
   - Parse product attributes from name
5. Write enriched CSV
6. Print output filename
7. Exit with appropriate code

## Files Modified/Created

### Created:
- **`python-scraper/enrich_models_cli.py`** - New CLI script for backend integration

### Modified:
- **`backend/src/routes/productsUpload.ts`** - Refactored `/products-fetch` endpoint

### Unchanged:
- **`python-scraper/enrich_models.py`** - Still works with CSV files for manual uploads

## Usage Examples

### Frontend Integration

```typescript
// Example React component usage
const fetchProducts = async (modelNumbers: string[]) => {
  const response = await fetch('/api/products-fetch', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${authToken}`
    },
    body: JSON.stringify({ modelNumbers })
  });

  const result = await response.json();
  
  console.log(`Created: ${result.summary.created}`);
  console.log(`Skipped: ${result.summary.skipped}`);
  console.log(`Failed: ${result.summary.failed}`);
  
  return result;
};

// Use it
await fetchProducts(['V5WG100LHL', 'V5GT110RHL']);
```

### Backend Testing (Direct API Call)

```bash
# Test with curl
curl -X POST http://localhost:8000/api/products-fetch \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "modelNumbers": ["V5WG100LHL", "V5GT110RHL"],
    "dryRun": true
  }'
```

### Python CLI Testing

```bash
# Direct Python script execution
cd python-scraper
.venv\Scripts\python.exe enrich_models_cli.py V5WG100LHL V5GT110RHL

# Output includes:
# - Detailed logs to stdout
# - OUTPUT_CSV:enriched_products_20251103_150524.csv
# - Exit code 0 on success
```

## Error Handling

### Backend Errors

1. **No model numbers**: Returns 400 with error message
2. **Python script not found**: Returns 500 with script path
3. **Python executable not found**: Returns 500 with executable path
4. **Python script timeout**: 5 minute timeout, returns 500
5. **Python script failure**: Returns 500 with exit code and logs
6. **No CSV generated**: Returns 500 with error message
7. **CSV parsing error**: Returns 500 with parse error details

### Python Script Errors

1. **No arguments**: Prints usage and exits with code 1
2. **Invalid model format**: Skips model and logs warning
3. **API capture failure**: Skips model and logs error
4. **No products found**: Exits with code 1
5. **WebDriver error**: Logs error and exits with code 1

## Dry Run Mode

Both backend and Python support dry run mode:

```json
{
  "modelNumbers": ["V5WG100LHL"],
  "dryRun": true
}
```

**Dry Run Behavior:**
- ✅ Python script executes normally
- ✅ Scrapes website and generates CSV
- ✅ Backend validates all data
- ❌ No database writes performed
- ✅ Returns what would be created

## Performance Considerations

### Timing Per Model:
- **Search + API capture**: ~2-3 seconds
- **Popup extraction**: ~3-4 seconds
- **Total per model**: ~5-7 seconds

### Batch Recommendations:
- **Optimal batch size**: 5-10 models
- **Max batch size**: 20 models (2-3 minutes)
- **Timeout**: 5 minutes (backend enforced)

### Scaling Strategies:
1. **Queue-based processing**: For large batches (50+ models)
2. **Background jobs**: Use job queue (Bull, etc.)
3. **Progress callbacks**: WebSocket updates to frontend
4. **Retry logic**: Automatic retry on failure

## Security Considerations

1. **Admin-only endpoint**: Requires `isAdmin: true` in JWT
2. **Input validation**: Validates modelNumbers array format
3. **Path traversal prevention**: Fixed paths for Python script/executable
4. **Timeout protection**: 5 minute hard limit
5. **Process isolation**: Python runs in separate process

## Monitoring & Debugging

### Backend Logs:
```typescript
console.log('products-fetch endpoint called');
console.log('Python process spawned successfully');
console.log('Python script completed successfully');
console.log('Transaction completed successfully');
```

### Python Logs:
```python
logging.info("[TIME] Starting fetch_product_details for model: X")
logging.info("[TIME] SUCCESS: Extracted prices from API: net=X, gross=Y")
logging.info("[TIME] SUCCESS: Extracted Einbau from popup: X-Y mm")
```

### Check Generated CSV:
```bash
# View generated CSV
cd python-scraper
ls -la enriched_products_*.csv

# View latest CSV
cat $(ls -t enriched_products_*.csv | head -1)
```

## Migration Guide

### From Old CSV-Based Flow:

**Before:**
1. Frontend sends modelNumbers
2. Backend creates temp CSV
3. Backend calls `enrich_models.py --input temp.csv`
4. Script reads CSV and processes

**After:**
1. Frontend sends modelNumbers
2. Backend calls `enrich_models_cli.py MODEL1 MODEL2 ...`
3. Script receives args directly and processes

**Benefits:**
- ✅ No temp CSV creation needed
- ✅ Simpler flow, fewer I/O operations
- ✅ Better error handling
- ✅ Direct argument passing

## Future Improvements

1. **Parallel processing**: Process multiple models concurrently
2. **Progress updates**: Real-time progress via WebSocket
3. **Caching**: Cache product details for duplicate requests
4. **Rate limiting**: Prevent abuse of scraping endpoint
5. **Queue system**: Handle large batches asynchronously

## Troubleshooting

### Issue: "Python script not found"
**Solution**: Verify path in backend matches actual location
```typescript
const scriptPath = path.join(process.cwd(), '..', 'python-scraper', 'enrich_models_cli.py');
```

### Issue: "No CSV generated"
**Solution**: Check Python logs for errors, ensure script completes successfully

### Issue: "Timeout after 5 minutes"
**Solution**: Reduce batch size or increase timeout limit

### Issue: "Products already exist"
**Solution**: Check `existingProducts` - use dry run first to validate

## Testing Checklist

- [ ] Frontend sends valid modelNumbers array
- [ ] Backend spawns Python process successfully
- [ ] Python script receives arguments correctly
- [ ] Python generates CSV with correct data
- [ ] Backend reads and parses CSV correctly
- [ ] Products created in database successfully
- [ ] Dry run mode works without database writes
- [ ] Error handling works for all failure cases
- [ ] Timeout protection works correctly
- [ ] Admin authorization enforced

## Support

For issues or questions:
1. Check backend logs: `console.log` output
2. Check Python logs: stdout/stderr from spawn
3. Verify generated CSV: `python-scraper/enriched_products_*.csv`
4. Test Python script directly: `python enrich_models_cli.py MODEL1`
5. Test with dry run first: `{ "dryRun": true }`
