# Bolt.DIY Deployment to Cloudflare Workers

This document records all commands and actions taken to clone, build, and deploy the Bolt.DIY project to Cloudflare Workers.

## 1. Repository Setup

### Clone the Repository
```bash
git clone https://github.com/teamxsparktest/bolt.diy2.git .
```

### Update Remote Repository URL
```bash
git remote set-url origin https://github.com/teamxsparktest/bolt.git
```

### Verify Remote URL
```bash
git config --get remote.origin.url
```

### Push Content to New Repository
```bash
git push -f origin main
```

## 2. Repository Changes

### Change Remote Repository
```bash
git remote remove origin
git remote add origin https://github.com/teamxsparktest/bolt.diy2.git
git fetch origin
```

### Check Remote Repository
```bash
git remote -v
```

## 3. Deploy to Cloudflare Workers

### Install Dependencies
```bash
pnpm install
```

### Create Worker Wrapper
Created a wrapper around the server build to make it compatible with Cloudflare Workers format:

```javascript
// worker/index.js
// Cloudflare Workers entry point
import { createRequestHandler } from "@remix-run/cloudflare";
import * as build from "../build/server/index.js";

// Create a fetch handler for Cloudflare Workers
export default {
  async fetch(request, env, ctx) {
    try {
      // Create a request handler using the server build
      const handleRequest = createRequestHandler({
        build,
        mode: "production",
      });

      // Handle the request
      return await handleRequest(request, env);
    } catch (error) {
      console.error("Error handling request:", error);
      return new Response("Server Error", { status: 500 });
    }
  }
};
```

### Update wrangler.toml Configuration
```toml
name = "bolt-diy"
main = "./worker/index.js"
compatibility_flags = ["nodejs_compat"]
compatibility_date = "2025-03-28"

[site]
bucket = "./build/client"

[build]
command = "pnpm run build"
```

### Build the Project
```bash
$env:NODE_OPTIONS="--max_old_space_size=4096"; pnpm run build
```
- Increased Node.js memory limit to 4GB to prevent "JavaScript heap out of memory" errors during build

### Deploy to Cloudflare Workers
```bash
wrangler deploy
```

## 4. Deployment Information
- **Project Name**: bolt-diy
- **Environment**: Production
- **Workers URL**: https://bolt-diy.roshanlama31769.workers.dev
- **Dashboard**: https://dash.cloudflare.com/e5d8e3e9586abd85a9d80d8c1a797ef6/workers/services/view/bolt-diy

## 5. Key Differences Between Pages and Workers Deployment

### Pages Deployment
- Primarily for static sites and frontend applications
- Uses `wrangler pages deploy` command
- Provides multiple deployment environments (preview/production)
- Simpler configuration

### Workers Deployment
- Full control over request handling
- Requires module format with default export
- Uses `wrangler deploy` command
- Accessible via workers.dev subdomain
- Can customize caching and other behaviors
