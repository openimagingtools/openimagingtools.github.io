# Webhook bridge: contribute form → GitHub Issues

Cloudflare Worker that receives direct form POSTs from openimagingtools.com/contribute
and creates a GitHub Issue for each submission, then redirects the visitor back
to /contribute/?submitted=true.

## Flow

```
openimagingtools.com/contribute (HTML form POST)
  → this Worker (parses fields, creates issue)
  → GitHub Issues API
  → redirect → /contribute/?submitted=true
```

## One-time setup

### 1. Create a GitHub fine-grained PAT

Go to **GitHub → Settings → Developer settings → Personal access tokens →
Fine-grained tokens → Generate new token**.

- Resource owner: `openimagingtools`
- Repository access: only `openimagingtools.github.io`
- Permissions:
  - Repository permissions → **Issues: Read and write**
  - Repository permissions → **Metadata: Read-only**

Copy the token — you'll need it in step 3.

### 2. Deploy the Worker

```bash
npm install -g wrangler
wrangler login
cd scripts/webhook-bridge
wrangler deploy
```

The Worker deploys to `https://openimagingtools-webhook-bridge.<subdomain>.workers.dev`.
Copy that URL from the deploy output — it becomes the form's `action`.

### 3. Add Worker secrets

```bash
wrangler secret put GITHUB_TOKEN
# paste the PAT from step 1

wrangler secret put GITHUB_REPO
# enter: openimagingtools/openimagingtools.github.io
```

### 4. Point the form at the Worker

In `contribute.html`, set the form `action` to the deployed Worker URL, and
update the success message to reference GitHub (submissions now become issues).

### 5. Test

Submit a test entry via `openimagingtools.com/contribute`. Verify:
- Browser redirects to `/contribute/?submitted=true` with the success message
- A GitHub Issue appears under the Issues tab with the correct title and label

## Updating the issue format

Edit `worker.js` and redeploy with `wrangler deploy`. No GitHub Actions involved —
the Worker talks to the GitHub Issues API directly.
