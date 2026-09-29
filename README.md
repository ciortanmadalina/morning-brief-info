# Morning Brief public information

Public homepage, privacy policy, and terms for a personal email automation. This repository contains no application code or credentials. The operator's contact email is intentionally public. Review the disclosures before publication.

Open `index.html` locally to preview. No build, dependencies, or server are needed. The allowlist in `.gitignore` helps prevent accidental additions; it is not a secret scanner and does not protect files added with `git add -f`.

## 1. Push this repository

On GitHub, create an empty **public** repository named `morning-brief-info` under `ciortanmadalina`. Do not initialize it with a README, license, or gitignore. Keep the separate `updates` application repository private.

In PowerShell:

```powershell
Set-Location C:\Users\ciort\ml\morning-brief-info
git status --short
git add .gitignore .nojekyll README.md index.html privacy.html terms.html styles.css
git diff --cached --stat
git commit -m "Add Morning Brief public information pages"
git push -u origin main
```

The local repository is initialized on `main`, with `origin` set to `https://github.com/ciortanmadalina/morning-brief-info.git`. No commit or push was made during setup. Never add token files, API keys, private config, email artifacts, or databases here.

## 2. Publish GitHub Pages

1. Open this repository's **Settings -> Pages**.
2. Choose **Deploy from a branch**.
3. Select **main** and **/(root)**, then Save. This repository does not use a `/docs` folder.
4. Wait for the Pages deployment to succeed in Actions.
5. Open all three links below in a private browser window. They must load without a GitHub login before you enter them in Google.

| Page | URL |
|---|---|
| Home | https://ciortanmadalina.github.io/morning-brief-info/ |
| Privacy | https://ciortanmadalina.github.io/morning-brief-info/privacy.html |
| Terms | https://ciortanmadalina.github.io/morning-brief-info/terms.html |

## 3. Complete Google Branding and publish OAuth

In Google Cloud, select the existing **Morning Brief** project. Do not create another OAuth client just to change publishing status.

1. Open **Google Auth Platform -> Branding**.
2. Keep app name **Morning Brief**, your support email, and developer contact email. Leave the logo blank.
3. Set **Application home page**, **Application privacy policy link**, and **Application terms of service link** to the matching URLs above.
4. Add **ciortanmadalina.github.io** to Authorized domains, without `https://` or a path.
5. Save, then open **Audience -> Publish app** and confirm. Check that the resulting Publishing status is **In production**. Publishing OAuth status is distinct from publishing or verifying branding.

If Google asks you to verify domain ownership, use Google Search Console with the same Google account that owns the Cloud project. For GitHub Pages, add the URL-prefix property `https://ciortanmadalina.github.io/morning-brief-info/` and select the HTML tag method. Add the exact `google-site-verification` meta tag supplied by Google inside the `<head>` of `index.html`, commit and push it, wait for Pages to redeploy, then click Verify. The verification tag is public by design; it is not an OAuth token. If Google requires verification at a different domain scope or rejects this property, stop and follow the exact requirement; do not claim ownership of `github.io` itself.

These pages do not guarantee Google publication or verification approval. Personal-use apps can be exempt from full OAuth review, but the console can still require configuration. If publishing remains disabled, inspect its specific validation message rather than adding fake URLs or submitting unnecessary scope verification.

In **Data Access**, ensure the requested Gmail scopes include both:

- `https://www.googleapis.com/auth/gmail.readonly`
- `https://www.googleapis.com/auth/gmail.send`

## 4. Obtain a new token after switching to Production

Keep Railway delivery disabled (`delivery.enabled = false` in `APP_CONFIG_TOML`), Cron Schedule empty, and Restart Policy Never while checking.

Run these commands from the **private application repository**, not this website repository:

```powershell
Set-Location C:\Users\ciort\ml\updates
uv run python main.py gmail-auth --include-send
uv run python main.py check gmail --include-send
```

Select the configured Gmail account and grant both read and send access. An unverified-app warning can be expected for your own personal-use project: continue only if the project and account are yours and recognized. Do not bypass organizational restrictions. These commands do not send email.

If authorization says it is local-only, the local process is seeing Railway/Google JSON environment variables. Use a local terminal without `RAILWAY_PROJECT_ID`, `GOOGLE_OAUTH_CLIENT_JSON`, or `GOOGLE_OAUTH_TOKEN_JSON`, and check whether your private `.env` defines them. Do not print or share their values. Use the local Desktop client credentials file for browser authorization.

After the local Gmail check passes, open the newly generated `token.json` privately in the application folder. In Railway Variables, replace the entire `GOOGLE_OAUTH_TOKEN_JSON` value with that file's complete JSON, with no extra quote wrapper. If you changed OAuth clients, also replace `GOOGLE_OAUTH_CLIENT_JSON` with the matching Desktop client JSON. Never put either value in this repository, screenshots, logs, or chat.

## 5. Validate Railway without sending

Apply the variables and use this temporary Start Command:

```sh
/bin/sh -c 'uv run --no-sync python main.py check config && uv run --no-sync python main.py check gmail --include-send && uv run --no-sync python main.py run-due --dry-run && uv run --no-sync python main.py status'
```

Deploy and inspect runtime logs. Expect configuration success, Gmail account/scope checks, schedule JSON, and a delivery-record count. A failure stops the chain. No email or model generation occurs. Keep the existing `/data` volume; do not delete duplicate-protection state to resolve an authorization error.

Production removes Google's Testing-mode seven-day refresh-token limit. It does not make tokens permanent: revocation, Gmail password changes, inactivity, token limits, or account policies can still invalidate them. Access tokens remain short-lived and the application refreshes them automatically.

## 6. Resume daily delivery only after checks pass

Inspect delivery state and resolve any interrupted or uncertain sends first. Complete the existing private application's deployment checks, including volume persistence and an accepted test email, before first activation. A real test can incur model costs and send email; it is not part of the diagnostics above.

For an already accepted deployment, restore these Railway settings together:

- Start Command: `uv run --no-sync python main.py run-due`
- Cron Schedule: `*/5 * * * *`
- One replica; Restart Policy Never.
- In `APP_CONFIG_TOML`: `delivery.enabled = true`, time `07:30`, timezone `Europe/Madrid`.

If enabled after today's cutoff and today's slot is unused, delivery can start immediately. The morning time is the preparation cutoff, not a guarantee of inbox arrival at exactly 07:30.

## References

- [Google refresh-token expiration](https://developers.google.com/identity/protocols/oauth2#expiration)
- [Google Branding settings](https://support.google.com/cloud/answer/15549049)
- [Personal-use verification exceptions](https://developers.google.com/identity/protocols/oauth2/production-readiness/brand-verification#personal-use)