# Apex Runner

A lightweight browser-based tool for running Apex scripts on Salesforce sandboxes. No server, no installation — just a single `index.html` on GitHub Pages.

## Why

When a new sandbox is created, a set of post-deployment Apex scripts needs to be executed — approval rules, CPQ configuration, permission sets, reference data, etc. Doing this manually in Developer Console (paste → execute → repeat) is slow and error-prone. Apex Runner automates the whole process.

## Features

- **One-click Salesforce login** — OAuth 2.0 PKCE with the public `PlatformCLI` client ID. No Connected App setup required.
- **Two script sources** — drag-and-drop a ZIP archive (no token needed) or read directly from a private GitHub repo.
- **Real-time execution** — scripts run sequentially via the Tooling API (`executeAnonymous`), results stream live.
- **Detailed error reporting** — compile errors, runtime exceptions with line numbers and error types.
- **Run history** — last 30 runs stored in the browser with pass/fail filtering and search.
- **HTML export** — download a standalone report for audit trails.
- **Built-in docs** — deployment guide, security model, FAQ — all inside the app.
- **Zero infrastructure** — static HTML file, talks directly to Salesforce and GitHub APIs from the browser.

## Quick Start

1. Create a GitHub repository (private recommended)
2. Place `index.html` in the root
3. Edit the `CONFIG` block at the top of the `<script>` section (see [Config Reference](#config-reference))
4. Enable GitHub Pages: **Settings → Pages → Source: Deploy from branch → main / root**
5. Open `https://<your-org>.github.io/<repo-name>/`, log in with Salesforce SSO, and run

## How It Works

### Step 1 — Connect

Enter your org's Instance URL and click **Login with Salesforce SSO**. You'll be redirected to the standard Salesforce login page (with your company's SSO button). After authenticating, you're returned to the app with a session token stored only in `sessionStorage` — it's gone when you close the tab.

### Step 2 — Select Scripts

Choose a source:

| | ZIP Archive | GitHub Repo |
|---|---|---|
| Requires token | No | Yes (PAT) |
| Always latest | Manual download | Always fresh |
| Works offline | Yes | No |

Select the folders and individual scripts you want to run. Scripts execute in alphabetical order within each folder — prefix filenames with numbers (`01_`, `02_`) to control order.

### Step 3 — Run

Each script is sent to Salesforce via the **Tooling API** (`executeAnonymous`) over HTTPS. Results appear in real time — pass ✅ or fail ❌ with error details and execution time. When finished, export a full HTML report.

## Script Repo Structure

```
scripts/
  apex/
    Post Deployment Scripts/        ← set this as DEFAULT_PATH
      Approval Rules/
        Credit Routers/
          CPQ1-168106.txt
          CPQ1-183625.txt
      CPQ Setup/
        01_price_rules.txt
        02_discount_schedules.txt
      Users/
        01_assign_permissions.txt
```

Both `.txt` and `.apex` extensions are supported by default.

## Config Reference

Edit the `CONFIG` object near the top of the `<script>` section in `index.html`:

| Field | Default | Description |
|---|---|---|
| `GH_TOKEN` | `''` | GitHub PAT for fetching scripts. Leave empty if users enter their own token in the UI. **Do not set in a public repo.** |
| `DEFAULT_REPO` | `''` | Default repository shown on step 2 (e.g. `'myorg/salesforce-scripts'`) |
| `DEFAULT_PATH` | `'scripts/apex/Post Deployment Scripts'` | Base path inside the repo where script folders live |
| `DEFAULT_EXTS` | `'.txt, .apex'` | Comma-separated file extensions to look for |

## Security

| Data | Storage | Scope |
|---|---|---|
| SF access token | `sessionStorage` | Current tab only. Gone on close. |
| GitHub PAT | `localStorage` | This browser on this device only. |
| Script contents | Browser memory | Sent to Salesforce via HTTPS only. |
| Instance URL | `localStorage` | Not a secret — public address. |

There is no backend. No data is logged, tracked, or stored anywhere except your own browser. All API calls go directly from your browser to Salesforce or GitHub over HTTPS. PKCE flow — no client secret required.

### GitHub Token Permissions

Use a **fine-grained PAT** with minimum permissions:
- Repository access: **Only select repositories** → scripts repo only
- Contents: **Read-only**

If your org enforces SAML SSO: after creating the token, click **Configure SSO → Authorize**.

## License

MIT
