# Function Timeout Notes for Netlify Deployment

## Current Timeout Configuration

### Netlify Limits
- **Free Tier:** 10 seconds maximum
- **Pro Tier:** 26 seconds maximum (configured in `netlify.toml`)

### Current API Route Timeouts

Routes with `maxDuration = 60` (18 routes):
- `/api/scrapewebsiteurl`
- `/api/scrapewebsitesubpages`
- `/api/scrapetwitterprofile`
- `/api/scrapereddit`
- `/api/scraperecenttweets`
- `/api/scrapelinkedin`
- `/api/findnews`
- `/api/findcompetitors`
- `/api/fetchyoutubevideos`
- `/api/fetchwikipedia`
- `/api/fetchtracxn`
- `/api/fetchtiktok`
- `/api/fetchpitchbook`
- `/api/fetchgithuburl`
- `/api/fetchfunding`
- `/api/fetchfounders`
- `/api/fetchfinancialreport`
- `/api/fetchcrunchbase`

Routes with `maxDuration = 100` (2 routes):
- `/api/companysummary` ⚠️ **Will timeout on Netlify**
- `/api/companymap` ⚠️ **Will timeout on Netlify**

## Recommendations

### Option 1: Upgrade to Netlify Pro
- Supports 26-second timeout
- Routes with `maxDuration = 60` will still timeout
- Routes with `maxDuration = 100` will still timeout

### Option 2: Reduce maxDuration Values

Update routes to match Netlify limits:

**For Pro tier (26 seconds):**
```typescript
export const maxDuration = 26;
```

**For Free tier (10 seconds):**
```typescript
export const maxDuration = 10;
```

### Option 3: Optimize Long-Running Operations

For routes that genuinely need more time:
1. Break operations into smaller chunks
2. Use background jobs with webhooks
3. Implement streaming responses
4. Cache results where possible

### Option 4: Use Netlify Background Functions

For operations exceeding timeout limits, consider using Netlify Background Functions (available on Pro tier):
- Can run up to 15 minutes
- Requires different implementation pattern
- Good for batch processing

## Quick Fix Script

To quickly update all routes to 26 seconds (Pro tier limit):

```bash
# Find all routes with maxDuration
find app/api -name "route.ts" -exec grep -l "maxDuration" {} \;

# Update all to 26 seconds
find app/api -name "route.ts" -exec sed -i '' 's/export const maxDuration = [0-9]*/export const maxDuration = 26/g' {} \;
```

## Testing Timeouts

After deployment, monitor function logs:
1. Go to Netlify Dashboard → Your Site → Functions
2. Check for timeout errors
3. Review execution times
4. Adjust `maxDuration` values accordingly

## Notes

- The `maxDuration` export in Next.js API routes is a hint, not a guarantee
- Netlify enforces hard limits regardless of `maxDuration` value
- Functions exceeding the limit will return a 500 error
- Consider implementing retry logic for critical operations

