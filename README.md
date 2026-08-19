# Mirror — setup instructions

## What's in this folder
- `index.html` — your website
- `api/claude.js` — the backend piece that talks to Claude safely (your API key never touches the browser)

## Step 5: Upload to GitHub
1. Go to github.com → click the **+** icon (top right) → **New repository**
2. Name it (e.g. `mirror-website`) → keep it **Public** or **Private**, either works → click **Create repository**
3. On the next page, click **uploading an existing file**
4. Drag in `index.html`, the `api` folder, and this `README.md`
5. Click **Commit changes**

## Step 6: Connect Vercel to GitHub
1. On your Vercel dashboard, click **Add New...** → **Project**
2. Find your `mirror-website` repo in the list → click **Import**
3. Leave all settings as default → click **Deploy**
4. Wait ~30 seconds — Vercel gives you a live link like `mirror-website.vercel.app`

## Step 7: Add your API key
1. In your Vercel project → **Settings** → **Environment Variables**
2. Name: `ANTHROPIC_API_KEY`
3. Value: paste your key from console.anthropic.com
4. Click **Save**
5. Go to the **Deployments** tab → click the **...** menu on the latest deployment → **Redeploy** (so it picks up the key)

## Step 8: Connect your domain
1. In Vercel → **Settings** → **Domains** → type your domain → **Add**
2. Vercel shows you 1–2 DNS records
3. Go to wherever you bought your domain → find **DNS settings** → add those records
4. Wait a bit (can take a few minutes to a few hours) — then your domain shows the live site
