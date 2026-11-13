# nOPV2 / bOPV2 Campaign — PHEM Data Entry

Daily campaign data entry for PHEM officers, served as a static site on GitHub
Pages and backed by the campaign's Google Apps Script project + Google Sheets.

- **Live site:** <https://aardmc.github.io/nOPV2_bOPV_Campaign/>
- **Campaign workbook:** `1_vn6o8K9p7TRog1Qv8f-FUzYvoaOwYRvJNBbvqOHIzg`
- **Officer credentials sheet:** `1Edg08nCZ-lbnsccrnhkFIWkZQzXSoLca90UNPjmofOs`
- **Apps Script project:** `1TNp7imIu78awjoqBRvyd5b-XPkkv1A4fymixQ4SaTqPH_4Fswg_FVQ9D`

## How it works

```
GitHub Pages (index.html)  ──HTTPS POST──▶  Apps Script doPost()  ──▶  Campaign workbook
        │                                          │
        └── signed session token ◀─────────────────┘
```

`index.html` is the original Apps Script UI with a small bridge added at the top
of the file. Inside Apps Script the bridge detects the native
`google.script.run` and does nothing, so the published Apps Script URL keeps
working exactly as before. On GitHub Pages the bridge provides
`google.script.run` by calling the JSON API in `Code.js` (`doPost`).

Signing in requires the four credentials from the officer list: **full name,
email, phone and Woreda ID**. The server returns a signed token (12 hours) that
is stored in `sessionStorage`; every later request is scoped to that officer's
Woreda ID.

## Officer experience

1. Open the site, enter name, email, phone and Woreda ID.
2. The app loads the officer's own woreda row from the `Day 1` plan.
3. Pick a campaign day, enter the numbers, press **Save**.
4. **Export my data (CSV)** downloads the officer's own woreda for all days.
5. **Logout** clears the session.

Officers can only read and write their own woreda row and only the input
columns (coverage, vials, alerts, places). Plan data, identifiers, settings and
formula columns are refused by the server.

The woreda bar at the top of the form is frozen while scrolling and collapses
to a slim bar that keeps the health center name in view. The site title bar
shows the Addis Ababa health bureau logo and the polio campaign badge on every
screen.

## Branding assets

| File | Use |
| --- | --- |
| `assets/logo.jpg` | Addis Ababa Regional Health Bureau logo (login card, title bar) |
| `assets/badge.png` | Polio vaccination campaign badge (login card, title bar, frozen bar, favicon) |
| `assets/favicon.png` | Browser tab icon, generated from the badge |

The images are served from GitHub Pages. The copy of the page inside Apps
Script (`Index.html`) must therefore reference them with absolute
`https://aardmc.github.io/nOPV2_bOPV_Campaign/assets/...` URLs — Apps Script
cannot serve files from this repository. Keep the relative `assets/...` paths
in this repo so the static site keeps working.

## Administrator features

The Google account that owns/deploys the Apps Script project is an
administrator automatically. To add other administrators, set the
`ADMIN_EMAILS` script property in the Apps Script project
(**Project Settings → Script properties**) to a comma-separated list of
emails:

```
ADMIN_EMAILS = admin@example.org, epi-lead@example.org
```

Administrators additionally see **Settings** (nOPV2/bOPV2 doses per vial,
revisit counting, wastage limit, campaign length) and the full-workbook
**Export Excel**. Everyone else gets the settings button hidden and a
woreda-scoped CSV export.

## Redeploying the Apps Script backend

The site is pointed at one Apps Script deployment. If the backend changes:

```bash
cd gas-provided
clasp push
clasp deploy -d "GitHub Pages API"
```

Then copy the new `/exec` URL into `window.CAMPAIGN_API_URL` at the top of
`index.html` and commit. Existing sessions keep working as long as the
`API_SIGNING_SECRET` script property is preserved (it is generated on first
use and stored in the project).

`clasp deployments` lists every deployment. The original deployment (`@1`) is
left untouched; the site uses the deployment created for the public API.

## Security notes

- No credentials, phone numbers or officer names are stored in this repository.
- The Apps Script endpoint is public (`ANYONE_ANONYMOUS`, runs as the deploying
  account) because the browser must reach it without a Google sign-in.
- Login attempts are throttled: 8 failures per email address per 15 minutes.
- Tokens are HMAC-SHA256 signed and expire after 12 hours; tampering or expiry
  returns `AUTH_REQUIRED`, and the page asks the officer to sign in again.
- Every read/write is validated against the officer's Woreda ID, the campaign
  day, the facility index and the writable column list.
- Secrets live in Apps Script script properties, never in this repository.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| "The campaign server is not responding" | Check the Apps Script deployment URL and that its access is *Anyone*. |
| "No account matches email + phone + Woreda ID" | The officer must use the email/phone/Woreda ID exactly as in the credentials sheet. |
| "Woreda not found in plan" | Add the woreda to `Day 1`, or correct the Woreda ID in the credentials sheet. |
| Settings button missing | Expected for non-administrators; add the email to `ADMIN_EMAILS`. |
| Session expired | Sign in again — tokens last 12 hours. |

## Local preview

The page is a single self-contained file; open it from any static server, e.g.:

```bash
npx serve .
```

GitHub Pages serves `index.html` from the repository root of `main`.
