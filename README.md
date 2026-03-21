# The Curiosity Engine — Deployment Guide

Two platforms, both free. Takes about 15 minutes total.

---

## What you're deploying

| File        | Where          | Purpose                              |
|-------------|----------------|--------------------------------------|
| `index.html`| GitHub Pages   | The quiz frontend (public website)   |
| `worker.js` | Cloudflare     | API proxy (keeps your key hidden)    |

---

## Step 1 — Deploy the Cloudflare Worker (proxy)

The worker hides your Anthropic API key. Users never see it.

### 1.1 Create a Cloudflare account
Go to https://dash.cloudflare.com and sign up (free, no credit card needed).

### 1.2 Create a new Worker
1. In the Cloudflare dashboard, click **Workers & Pages** in the left sidebar
2. Click **Create** → **Create Worker**
3. Give it a name, e.g. `curiosity-quiz-proxy`
4. Click **Deploy** (ignore the placeholder code for now)

### 1.3 Paste the worker code
1. Click **Edit code** on your new worker
2. Delete all existing code
3. Paste the entire contents of `worker.js`
4. Click **Save and deploy**

Your worker URL will look like:
```
https://curiosity-quiz-proxy.YOUR_SUBDOMAIN.workers.dev
```
Copy this URL — you'll need it in Step 3.

### 1.4 Add your Anthropic API key as a secret
1. Go to your worker → **Settings** → **Variables**
2. Under **Environment Variables**, click **Add variable**
3. Set **Variable name** to: `ANTHROPIC_API_KEY`
4. Set **Value** to your key (starts with `sk-ant-...`)
5. Click the **Encrypt** toggle so it becomes a secret
6. Click **Save and deploy**

Get a key at: https://console.anthropic.com

---

## Step 2 — Deploy the frontend to GitHub Pages

### 2.1 Create a GitHub account
Go to https://github.com and sign up if you don't have one.

### 2.2 Create a new repository
1. Click the **+** icon → **New repository**
2. Name it: `curiosity-quiz` (or any name you like)
3. Set it to **Public**
4. Check **Add a README file**
5. Click **Create repository**

### 2.3 Upload index.html
1. In your new repository, click **Add file** → **Upload files**
2. Drag and drop `index.html`
3. Click **Commit changes**

### 2.4 Enable GitHub Pages
1. Go to **Settings** → **Pages** (in the left sidebar)
2. Under **Source**, select **Deploy from a branch**
3. Choose branch: `main`, folder: `/ (root)`
4. Click **Save**

Your site URL will be:
```
https://YOUR_GITHUB_USERNAME.github.io/curiosity-quiz
```
It takes about 1–2 minutes to go live.

---

## Step 3 — Connect the two together

Now you need to tell:
- The **frontend** where the worker lives
- The **worker** which frontend is allowed to call it

### 3.1 Update index.html — set PROXY_URL
Open `index.html` and find this line near the bottom:

```js
const PROXY_URL = 'https://YOUR_WORKER_SUBDOMAIN.workers.dev/quiz';
```

Replace it with your actual worker URL, e.g.:
```js
const PROXY_URL = 'https://curiosity-quiz-proxy.myname.workers.dev/quiz';
```

Re-upload the updated `index.html` to your GitHub repository.

### 3.2 Update worker.js — set ALLOWED_ORIGIN
Open `worker.js` and find this line near the top:

```js
const ALLOWED_ORIGIN = 'https://YOUR_GITHUB_USERNAME.github.io';
```

Replace it with your actual GitHub Pages URL, e.g.:
```js
const ALLOWED_ORIGIN = 'https://vishrut.github.io';
```

Paste the updated code back into the Cloudflare Worker editor and click **Save and deploy**.

---

## Step 4 — Test it

Visit your GitHub Pages URL and try starting a quiz. If it works, you're live.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| "Could not reach the server" | PROXY_URL wrong in index.html | Double-check the URL matches your worker |
| Worker returns 401 | API key not saved correctly | Re-add ANTHROPIC_API_KEY secret in Cloudflare |
| CORS error in browser console | ALLOWED_ORIGIN mismatch | Make sure worker ALLOWED_ORIGIN matches your GitHub Pages domain exactly |
| GitHub Pages shows 404 | Pages not enabled or file not uploaded | Check Settings → Pages in GitHub |

---

## Free tier limits

| Platform | Free allowance | Notes |
|----------|---------------|-------|
| GitHub Pages | Unlimited | Static hosting, no limits |
| Cloudflare Workers | 100,000 requests/day | More than enough for personal use |
| Anthropic API | Pay-per-use | ~$0.003 per quiz round on Claude Sonnet |

---

## Custom domain (optional)

If you own a domain (e.g. `curiosityquiz.com`):
1. In GitHub Pages settings, enter it under **Custom domain**
2. Update your DNS with a CNAME record pointing to `YOUR_USERNAME.github.io`
3. Update `ALLOWED_ORIGIN` in the worker to match the new domain
