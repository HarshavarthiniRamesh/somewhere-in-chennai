# Collecting form submissions in Google Sheets

Submissions land in a spreadsheet you check by hand. Nothing reaches the site
automatically — `src/data/communities.js` is still the only source the site
reads, and you edit it yourself once you've vetted an entry.

## Setup, once

1. **New Google Sheet.** Call it whatever you like. Put these headers in row 1,
   spelled exactly:

   ```
   timestamp | name | description | entry | social | status
   ```

   `status` is yours to fill in — `new`, `added`, `rejected`.

2. **Extensions → Apps Script.** Delete the placeholder and paste:

   ```js
   function doPost(e) {
     const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Sheet1');
     const f = e.parameter;
     sheet.appendRow([
       new Date(),
       f.name || '',
       f.description || '',
       f.entry || '',
       f.social || '',
       'new',
     ]);
     return ContentService.createTextOutput('ok');
   }
   ```

3. **Deploy → New deployment → Web app.**
   - Execute as: **Me**
   - Who has access: **Anyone**

   "Anyone" is required — the browser posts without a Google login. It only
   exposes `doPost`, which appends a row; nobody can read the sheet through it.

4. Copy the deployment URL and set it at the top of the inline script in
   `src/pages/index.astro`:

   ```js
   const SHEET_ENDPOINT = "https://script.google.com/macros/s/..../exec";
   ```

That's the whole change — the POST is already written and runs as soon as the
constant is non-empty.

## Why it's built this way

The form posts `FormData` with `mode: "no-cors"`. That sidesteps Apps Script's
CORS behaviour, at the cost of the response being opaque: the browser can't
read the status code, so the site says "thank you" whether or not the row was
written. Worth testing the endpoint once after deploying.

Field names map straight across: `name`, `description`, `entry`, `social`.
`entry` arrives pre-joined as `"Free, Paid"` from the multiselect.

## Worth knowing

- **Re-deploy after editing the script.** Apps Script serves the last deployed
  version, not the saved one. Use *Manage deployments → edit → New version*, or
  the URL keeps running the old code.
- **If the URL answers "You need access" (HTTP 403), make a new deployment.**
  This happened on first setup: the deployment showed *Who has access: Anyone*
  in the editor but still rejected anonymous requests, and editing it in place
  didn't help. *Deploy → New deployment → Web app → Anyone* produced a fresh
  URL that worked straight away. Archive the old one. A quick check from the
  terminal — a 302 to `script.googleusercontent.com` means it ran, `ok` is the
  script's reply:

  ```sh
  curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' -X POST \
    -F name=Test -F description=x -F entry=Free -F social=x "$SHEET_ENDPOINT"
  ```
- **No spam protection.** A public endpoint will eventually get junk. If it
  becomes a problem, the cheap fixes are a honeypot field (a hidden input real
  people leave empty, rejected server-side) or a rate check on timestamp.
- **Set up a notification** so you notice submissions: in the Sheet,
  *Tools → Notification settings → Notify me when any changes are made*.

## Alternatives, if Apps Script annoys you

- **Formspree / Basin / Formcarry** — paste an endpoint, they email you and keep
  a dashboard. Free tiers cover this volume. No script to maintain, but a
  third party holds the data.
- **Netlify Forms** — if the site ends up on Netlify, add `netlify` to the form
  tag and submissions appear in the dashboard. Zero code, but it ties
  submissions to the host.

Apps Script is the better fit here: the data lands somewhere you already own,
and a spreadsheet is a good place to triage a queue you're vetting by hand.
