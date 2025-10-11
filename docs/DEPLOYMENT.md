# Deployment Guide

This guide covers deployment options for the Gemini 2.0 Playground.

## Table of Contents

1. [Deployment Options](#deployment-options)
2. [Deno Deploy (Recommended)](#deno-deploy-recommended)
3. [Cloudflare Workers](#cloudflare-workers)
4. [Local Development](#local-development)
5. [Configuration](#configuration)
6. [Custom Domain Setup](#custom-domain-setup)
7. [Troubleshooting](#troubleshooting)

---

## Deployment Options

### Comparison

| Feature | Deno Deploy | Cloudflare Workers | Local |
|---------|-------------|-------------------|-------|
| **Setup Time** | ~2 minutes | ~5 minutes | ~1 minute |
| **Free Tier** | Yes | Yes | N/A |
| **Custom Domain** | Optional | Required (China) | N/A |
| **HTTPS** | Automatic | Automatic | Manual |
| **Global CDN** | Yes | Yes | No |
| **TypeScript** | Native | Via build | Native |
| **China Access** | Direct | Via custom domain | Depends |

### Recommendations

- **For Quick Start**: Deno Deploy
- **For Production**: Deno Deploy or Cloudflare Workers
- **For Development**: Local with Deno
- **For China Users**: Deno Deploy (easier) or Cloudflare Workers with custom domain

---

## Deno Deploy (Recommended)

### Prerequisites

- GitHub account
- Gemini API key from [https://aistudio.google.com](https://aistudio.google.com)

### Step-by-Step Deployment

#### 1. Fork the Repository

1. Go to [https://github.com/tech-shrimp/gemini-playground](https://github.com/tech-shrimp/gemini-playground)
2. Click the "Fork" button in the top right
3. Wait for the fork to complete

#### 2. Create Deno Deploy Project

1. Visit [https://dash.deno.com/](https://dash.deno.com/)
2. Sign in with GitHub
3. Click "New Project"
4. Select your forked repository
5. Configure the project:

```
Repository: YOUR_USERNAME/gemini-playground
Branch: main
Entrypoint: src/deno_index.ts
Production Branch: main
```

6. Click "Deploy Project"

#### 3. Access Your Deployment

1. Wait for deployment to complete (~30 seconds)
2. Your app will be available at: `https://YOUR_PROJECT_NAME.deno.dev`
3. Open the URL and enter your Gemini API key
4. Click "Connect" to start using the app

### Automatic Deployments

Once set up, Deno Deploy automatically deploys on every push to your repository's main branch.

### Environment Variables (Optional)

While not required (API key is entered in the UI), you can set environment variables:

1. Go to your project settings
2. Navigate to "Environment Variables"
3. Add any custom configuration

### Custom Domain Setup

1. Go to project settings
2. Click "Domains"
3. Click "Add Domain"
4. Follow DNS configuration instructions

---

## Cloudflare Workers

### Prerequisites

- Cloudflare account
- GitHub account
- Gemini API key
- Cloudflare API token
- Custom domain (for China access)

### Step-by-Step Deployment

#### 1. Get Cloudflare Credentials

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Go to "My Profile" → "API Tokens"
3. Create token with "Edit Cloudflare Workers" permissions
4. Note your Account ID (from Workers dashboard)

#### 2. Deploy via Button

1. Click the deploy button:
   [![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/tech-shrimp/gemini-playground)

2. Authorize GitHub access
3. Enter Cloudflare credentials:
   - Account ID
   - API Token
4. Click "Deploy"

#### 3. Enable GitHub Actions

1. Go to your forked repository on GitHub
2. Click "Actions" tab
3. Click "I understand my workflows, enable them"

#### 4. Configure Worker

1. Go to [Cloudflare Workers Dashboard](https://dash.cloudflare.com)
2. Find your deployed worker
3. Click "Settings"
4. Configure as needed

#### 5. Bind Custom Domain

**Important for China users:**

1. Click "Triggers" tab
2. Click "Add Custom Domain"
3. Enter your domain (e.g., `gemini.yourdomain.com`)
4. Follow DNS setup instructions
5. Wait for SSL certificate provisioning

### Manual Deployment

If you prefer manual deployment:

```bash
# Install Wrangler CLI
npm install -g wrangler

# Authenticate
wrangler login

# Configure wrangler.toml
# Edit account_id in wrangler.toml

# Deploy
npm run deploy
```

### wrangler.toml Configuration

```toml
name = "gemini-playground"
main = "src/index.js"
compatibility_date = "2024-01-01"

[site]
bucket = "src/static"

[build]
command = "echo 'No build step required'"

[env.production]
workers_dev = false
route = "your-domain.com/*"
```

---

## Local Development

### Deno Development

#### 1. Install Deno

**Windows:**
```powershell
irm https://deno.land/install.ps1 | iex
```

**macOS/Linux:**
```bash
curl -fsSL https://deno.land/install.sh | sh
```

#### 2. Run the Server

```bash
# Clone repository
git clone https://github.com/tech-shrimp/gemini-playground.git
cd gemini-playground

# Start server
deno run --allow-net --allow-read src/deno_index.ts
```

#### 3. Access Locally

Open [http://localhost:8000](http://localhost:8000)

### Cloudflare Workers Development

#### 1. Install Dependencies

```bash
npm install
```

#### 2. Start Development Server

```bash
npm run dev
# or
npm start
```

#### 3. Access Locally

Open [http://localhost:8787](http://localhost:8787)

### Development Tips

```bash
# Watch for changes (Deno)
deno run --allow-net --allow-read --watch src/deno_index.ts

# Debug mode
deno run --allow-net --allow-read --inspect src/deno_index.ts

# Run tests
npm test
```

---

## Configuration

### Client Configuration

Edit `src/static/js/config/config.js`:

```javascript
export const CONFIG = {
  API: {
    VERSION: 'v1alpha',
    MODEL_NAME: 'models/gemini-2.0-flash-exp'
  },
  SYSTEM_INSTRUCTION: {
    TEXT: 'You are my helpful assistant...'
  },
  AUDIO: {
    SAMPLE_RATE: 16000,
    OUTPUT_SAMPLE_RATE: 24000,
    BUFFER_SIZE: 2048,
    CHANNELS: 1
  }
};
```

### Available Models

To use different models, update `MODEL_NAME`:

```javascript
// Latest experimental model
MODEL_NAME: 'models/gemini-2.0-flash-exp'

// Stable model
MODEL_NAME: 'models/gemini-1.5-pro-latest'

// Learning model
MODEL_NAME: 'models/learnlm-1.5-pro-experimental'
```

### Voice Configuration

Available voices for audio responses:

```javascript
// In main.js or via UI dropdown
voiceSelect.value = 'Aoede';  // Default
// Options: Aoede, Charon, Fenrir, Kore, Puck
```

---

## Custom Domain Setup

### Deno Deploy Custom Domain

1. **DNS Configuration:**
   ```
   Type: CNAME
   Name: gemini (or your subdomain)
   Value: cname.deno.dev
   TTL: Auto
   ```

2. **Add to Deno:**
   - Go to project settings
   - Click "Domains"
   - Add your domain
   - Wait for SSL provisioning

3. **Verification:**
   - DNS propagation: up to 24 hours
   - SSL certificate: ~5 minutes
   - Test: `https://your-domain.com`

### Cloudflare Workers Custom Domain

1. **Add Domain to Cloudflare:**
   - Add site to Cloudflare account
   - Update nameservers at registrar

2. **Configure Worker Route:**
   - Go to Worker settings
   - Add route: `your-domain.com/*`
   - Enable SSL

3. **China-specific Configuration:**
   ```
   - Domain must be ICP licensed (if in China)
   - Use Cloudflare China network if available
   - Test from China location
   ```

---

## Production Checklist

### Before Deploying

- [ ] Test locally with all features
- [ ] Update system instructions if needed
- [ ] Configure custom domain (if required)
- [ ] Test API key validation
- [ ] Verify HTTPS is enforced
- [ ] Test from target regions

### After Deploying

- [ ] Test WebSocket connection
- [ ] Test audio recording/playback
- [ ] Test video capture
- [ ] Test screen sharing
- [ ] Test tool execution
- [ ] Monitor error logs
- [ ] Check performance metrics

### Security Checklist

- [ ] HTTPS enabled
- [ ] CORS properly configured
- [ ] No API keys in code
- [ ] No sensitive data logging
- [ ] Rate limiting considered
- [ ] Content Security Policy set

---

## Monitoring

### Deno Deploy Monitoring

1. **Dashboard Metrics:**
   - Go to project dashboard
   - View request count, latency, errors
   - Check deployment history

2. **Logs:**
   ```bash
   # View logs
   deployctl logs --project=YOUR_PROJECT_NAME
   ```

### Cloudflare Workers Monitoring

1. **Analytics:**
   - Workers dashboard → Analytics
   - View requests, errors, CPU time

2. **Logs:**
   - Real-time logs in dashboard
   - Use `wrangler tail` for local viewing

3. **Alerts:**
   - Set up alerts for errors
   - Monitor rate limits

---

## Troubleshooting

### Common Issues

#### 1. WebSocket Connection Fails

**Symptoms:** "Could not connect" error

**Solutions:**
```bash
# Check deployment logs
deployctl logs --project=YOUR_PROJECT_NAME

# Verify API key is correct
# Check browser console for errors
# Try different network (VPN if needed)
```

#### 2. Audio Not Working

**Symptoms:** No audio playback or recording

**Solutions:**
```javascript
// Check browser permissions
navigator.permissions.query({name: 'microphone'})

// Check AudioContext state
console.log(audioContext.state); // Should be 'running'

// Resume context on user interaction
audioContext.resume();
```

#### 3. Video Not Capturing

**Symptoms:** Black screen or permission denied

**Solutions:**
```javascript
// Check browser support
VideoRecorder.checkBrowserSupport();

// Check HTTPS (required for camera)
console.log(location.protocol); // Should be 'https:'

// Request permissions
navigator.mediaDevices.getUserMedia({video: true})
```

#### 4. China Access Issues

**Cloudflare Workers:**
```
Error: "User location is not supported"
Solution: Use custom domain
```

**Deno Deploy:**
```
Generally works directly
If issues: Try different edge location
```

#### 5. High Latency

**Symptoms:** Slow responses

**Solutions:**
```bash
# Check network latency
ping generativelanguage.googleapis.com

# Optimize video FPS
videoManager.start(5, callback); // Lower FPS

# Reduce audio quality if needed
```

### Debug Mode

Enable detailed logging:

```javascript
// In main.js
Logger.getInstance().on('log', (entry) => {
  console.log(`[${entry.level}] ${entry.message}`, entry.data);
});

// WebSocket debugging
client.on('log', (log) => {
  console.log(`[WS] ${log.type}:`, log.message);
});
```

### Getting Help

1. **Check Browser Console:**
   - F12 → Console tab
   - Look for errors

2. **Export Logs:**
   ```javascript
   Logger.export(); // Downloads logs
   ```

3. **GitHub Issues:**
   - [Create an issue](https://github.com/tech-shrimp/gemini-playground/issues)
   - Include logs and error messages

4. **Community:**
   - Check existing issues
   - Search documentation

---

## Performance Optimization

### Client-Side

```javascript
// Reduce video frame rate
const fps = 10; // Lower = less bandwidth

// Adjust JPEG quality
const quality = 0.5; // Lower = smaller files

// Limit video resolution
const width = 320;
const height = 240;
```

### Network Optimization

```javascript
// For slow connections:
CONFIG.AUDIO.SAMPLE_RATE = 8000; // Lower quality
videoManager.start(5, callback);  // Lower FPS
```

### Server-Side

```javascript
// Cloudflare Workers: Enable caching for static assets
// Deno Deploy: Automatic edge caching
```

---

## Backup and Recovery

### Backup Configuration

```bash
# Backup your fork
git clone https://github.com/YOUR_USERNAME/gemini-playground.git
cd gemini-playground
git remote add upstream https://github.com/tech-shrimp/gemini-playground.git
```

### Update from Upstream

```bash
# Get latest changes
git fetch upstream
git merge upstream/main

# Push to your fork
git push origin main

# Deployment will auto-update
```

### Rollback Deployment

**Deno Deploy:**
1. Go to Deployments tab
2. Find previous working deployment
3. Click "Promote to Production"

**Cloudflare Workers:**
```bash
wrangler rollback
```

---

## Advanced Deployment

### Multi-Environment Setup

```bash
# Development
deno run --allow-net --allow-read src/deno_index.ts

# Staging
deployctl deploy --project=gemini-staging src/deno_index.ts

# Production
deployctl deploy --project=gemini-prod src/deno_index.ts
```

### CI/CD Pipeline

GitHub Actions example (`.github/workflows/deploy.yml`):

```yaml
name: Deploy to Deno
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: denoland/setup-deno@v1
      - run: deno test
      - uses: denoland/deployctl@v1
        with:
          project: gemini-playground
          entrypoint: src/deno_index.ts
          token: ${{ secrets.DENO_DEPLOY_TOKEN }}
```

### Load Balancing

Both platforms automatically handle:
- Global edge distribution
- Automatic failover
- Geographic routing
- DDoS protection

---

## Cost Considerations

### Free Tiers

**Deno Deploy:**
- 100,000 requests/day
- 100 GB data transfer/month
- No credit card required

**Cloudflare Workers:**
- 100,000 requests/day
- 10ms CPU time per request
- No credit card required

### Paid Plans

**Deno Deploy Pro:**
- $20/month
- 5M requests/month
- 100 GB data transfer/month

**Cloudflare Workers Paid:**
- $5/month base
- $0.50 per million requests

### Gemini API Costs

Check [Google AI Studio](https://aistudio.google.com) for:
- Free tier limits
- Pricing per token
- Rate limits

---

## Next Steps

After deployment:

1. **Test all features** - Verify everything works
2. **Set up monitoring** - Track usage and errors
3. **Configure custom domain** (optional)
4. **Customize UI** - Brand it for your needs
5. **Add analytics** (optional)
6. **Set up backups** - Keep configuration safe

---

## Additional Resources

- [Deno Deploy Documentation](https://deno.com/deploy/docs)
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
- [Gemini API Documentation](https://ai.google.dev/docs)
- [Project GitHub](https://github.com/tech-shrimp/gemini-playground)

---

This deployment guide should help you get the Gemini 2.0 Playground up and running on your preferred platform. For additional help, check the GitHub issues or create a new issue with your deployment questions.
