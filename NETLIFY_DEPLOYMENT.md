# Netlify Deployment Guide for Company Researcher

This guide will help you deploy the Company Researcher application to Netlify instead of Vercel.

## Changes Made for Netlify Compatibility

1. ✅ **Removed Vercel Analytics** - The `@vercel/analytics` package has been removed as it's Vercel-specific
2. ✅ **Added `netlify.toml`** - Configuration file for Netlify deployment
3. ✅ **Next.js Plugin** - Uses `@netlify/plugin-nextjs` for optimal Next.js support

## Prerequisites

- A Netlify account ([sign up here](https://app.netlify.com/signup))
- A GitHub/GitLab/Bitbucket repository
- Your API keys:
  - Exa API key ([get it here](https://dashboard.exa.ai/api-keys))
  - Anthropic API key ([get it here](https://console.anthropic.com/))
  - (Optional) YouTube API key
  - (Optional) GitHub token

## Deployment Steps

### Method 1: Git Integration (Recommended)

1. **Push your code to Git**
   ```bash
   git add .
   git commit -m "Configure for Netlify deployment"
   git push origin main
   ```

2. **Connect to Netlify**
   - Go to [Netlify Dashboard](https://app.netlify.com)
   - Click **"Add new site"** → **"Import an existing project"**
   - Choose your Git provider (GitHub, GitLab, or Bitbucket)
   - Authorize Netlify to access your repositories
   - Select the `company-researcher` repository

3. **Configure Build Settings**
   
   Netlify should auto-detect Next.js from `netlify.toml`:
   - **Build command:** `npm run build` (auto-detected)
   - **Publish directory:** `.next` (auto-detected)
   - **Node version:** 18 (configured in `netlify.toml`)

   If not auto-detected, manually enter:
   - Build command: `npm run build`
   - Publish directory: `.next`

4. **Set Environment Variables**
   
   Go to **Site settings** → **Environment variables** and add:
   
   | Variable Name | Description |
   |--------------|-------------|
   | `EXA_API_KEY` | Your Exa API key |
   | `ANTHROPIC_API_KEY` | Your Anthropic API key |
   | `YOUTUBE_API_KEY` | (Optional) YouTube API key |
   | `NEXT_PUBLIC_GITHUB_TOKEN` | (Optional) GitHub token |

5. **Deploy**
   - Click **"Deploy site"**
   - Wait for the build to complete (typically 3-5 minutes for first build)
   - Your site will be available at `https://your-site-name.netlify.app`

### Method 2: Netlify CLI

1. **Install Netlify CLI**
   ```bash
   npm install -g netlify-cli
   ```

2. **Login to Netlify**
   ```bash
   netlify login
   ```

3. **Initialize Site**
   ```bash
   netlify init
   ```
   
   Follow the prompts:
   - Create a new site or link to existing site
   - Build command: `npm run build`
   - Publish directory: `.next`

4. **Set Environment Variables**
   ```bash
   netlify env:set EXA_API_KEY "your-exa-api-key"
   netlify env:set ANTHROPIC_API_KEY "your-anthropic-api-key"
   # Optional
   netlify env:set YOUTUBE_API_KEY "your-youtube-api-key"
   netlify env:set NEXT_PUBLIC_GITHUB_TOKEN "your-github-token"
   ```

5. **Deploy**
   ```bash
   # Deploy to production
   netlify deploy --prod
   
   # Or deploy a draft/preview first
   netlify deploy
   ```

6. **Test Locally with Netlify**
   ```bash
   netlify dev
   ```
   
   This runs your site locally at `http://localhost:8888` with Netlify's serverless functions simulation.

## Important Notes

### Function Timeout Limits

⚠️ **Timeout Considerations:**

- **Free tier:** 10-second timeout
- **Pro tier:** 26-second timeout (configured in `netlify.toml`)

Some API routes have `maxDuration` set to 60 or 100 seconds:
- Routes with `maxDuration = 60`: May timeout on free tier, should work on Pro tier
- Routes with `maxDuration = 100`: Will timeout on both tiers

**Routes that may need attention:**
- `/api/companysummary` - `maxDuration = 100` ⚠️
- `/api/companymap` - `maxDuration = 100` ⚠️
- Most other routes - `maxDuration = 60` ⚠️ (free tier)

**Solutions:**
1. **Upgrade to Netlify Pro** - Supports 26-second timeout
2. **Optimize API routes** - Break down long-running operations
3. **Use background jobs** - For operations exceeding timeout limits
4. **Adjust `maxDuration`** - Set to 26 or less for Pro tier compatibility

### Next.js Plugin

The `@netlify/plugin-nextjs` plugin is automatically installed during build. It:
- Converts Next.js API routes to Netlify Functions
- Handles routing automatically
- Optimizes static assets
- Supports ISR (Incremental Static Regeneration)

### Analytics

Vercel Analytics has been removed. For analytics on Netlify:
- Use **Netlify Analytics** (available on Pro tier)
- Or integrate a third-party solution like Google Analytics, Plausible, etc.

## Troubleshooting

### Build Fails

**Issue:** Build timeout or errors
**Solution:**
- Check build logs in Netlify dashboard
- Verify Node version is 18+
- Ensure all dependencies install correctly
- Check for TypeScript/ESLint errors

### API Routes Not Working

**Issue:** 404 errors on `/api/*` routes
**Solution:**
- Verify `@netlify/plugin-nextjs` is installed (auto-installed)
- Check that API routes are in `app/api/` directory
- Verify function timeouts are sufficient

### Function Timeout Errors

**Issue:** API routes timing out
**Solution:**
- Check function logs in Netlify Dashboard → Functions
- Reduce `maxDuration` values in API routes
- Consider upgrading to Pro tier for 26-second timeout
- Optimize long-running operations

### Environment Variables Not Working

**Issue:** Variables undefined in runtime
**Solution:**
- Verify variables are set in Netlify dashboard
- Check variable names match exactly (case-sensitive)
- Redeploy after adding variables
- Check scope settings (production vs preview)

## Performance Optimization

### Build Optimization

- Netlify automatically caches `node_modules` between builds
- Use `.netlify` directory for custom cache configuration
- Monitor build logs to identify slow steps

### Runtime Optimization

- API routes run as serverless functions
- Static pages are pre-rendered and cached
- Dynamic routes are rendered on-demand

## Cost Considerations

- **Free tier includes:**
  - 100 GB bandwidth/month
  - 300 build minutes/month
  - 125,000 serverless function invocations/month
  - 10-second function timeout

- **Pro tier ($19/month) includes:**
  - Everything in free tier
  - 26-second function timeout
  - Netlify Analytics
  - Priority support

## Custom Domain

1. Go to **Domain settings** → **Add custom domain**
2. Enter your domain name
3. Follow DNS configuration instructions
4. Netlify provides free SSL certificates automatically

## Continuous Deployment

With Git integration enabled:
- **Main branch:** Deploys to production
- **Pull requests:** Create preview deployments
- **Other branches:** Can be configured for branch-specific deployments

## Support

- [Netlify Documentation](https://docs.netlify.com)
- [Next.js on Netlify](https://docs.netlify.com/integrations/frameworks/next-js/)
- [Netlify Support](https://www.netlify.com/support)

## Migration Checklist

- [x] Remove Vercel Analytics
- [x] Create `netlify.toml` configuration
- [x] Update documentation
- [ ] Review function timeout requirements
- [ ] Set environment variables in Netlify
- [ ] Test deployment
- [ ] Verify all API routes work correctly
- [ ] Set up custom domain (optional)
- [ ] Configure analytics (optional)

