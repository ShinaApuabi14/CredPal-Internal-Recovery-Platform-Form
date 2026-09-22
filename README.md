# CredPal Recovery Engagement Log — Setup Guide

This gives you a Google Sheet-backed HTML form for recovery agents to log
engagements against accounts. Two files:

- **Code.gs** — Apps Script backend (lives inside the Google Sheet)
- **recovery-engagement-form.html** — the form agents fill in (host it
  anywhere, or just open it in a browser)

The dropdown options and every submitted entry live in the Sheet, so it can
be edited from either side: agents submit through the form, and you (or
anyone with edit access) can add, correct, or bulk-edit rows directly in the
Sheet — including changing the dropdown option lists at any time.

## 1. Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new
   blank spreadsheet.
2. Name it something like **"Recovery Engagement Log"**.

## 2. Add the Apps Script backend

1. In the Sheet, go to **Extensions > Apps Script**.
2. Delete any starter code in the editor.
3. Open **Code.gs** from this folder, copy its entire contents, and paste it
   into the Apps Script editor.
4. Click the disk icon (or Ctrl/Cmd+S) to save. Name the project if asked
   (e.g. "Recovery Engagement Backend").

## 3. Run the one-time setup

1. In the Apps Script editor toolbar, make sure the function selector
   dropdown (next to "Debug") shows **setupSheet**.
2. Click **Run**.
3. The first time, Google will ask you to authorize the script — click
   **Review permissions**, choose your account, click **Advanced** if it
   warns about an unverified app, then **Go to (project name)** and
   **Allow**. This is expected for your own scripts.
4. Go back to the Sheet — you should now see two tabs:
   - **Entries** — where every form submission lands
   - **Lists** — dropdown option lists, pre-filled with starter values
     (Recovery Agent, Engagement Channel, Engagement Status, Response,
     Payment Reported, Plan Type, Changes Current Arrangement)
5. Edit the **Lists** tab now with your real recovery agent names and any
   option changes you want — the form reads this tab live, and the Sheet's
   own dropdown cells in Entries also validate against it.

You can re-run `setupSheet` any time (e.g. after a script update) — it only
fills in what's missing and won't wipe existing data or list values.

## 4. Deploy as a Web App

1. In the Apps Script editor, click **Deploy > New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Set:
   - **Execute as:** Me (your account)
   - **Who has access:** Anyone
4. Click **Deploy**, authorize again if prompted.
5. Copy the **Web app URL** shown (it looks like
   `https://script.google.com/macros/s/XXXXXXXXXXXX/exec`).

Keep this URL private-ish — anyone with it can submit entries (there's no
login on the form). If you ever need to lock it down further, add a shared
secret check in `doPost` or switch "Who has access" to your Google
Workspace domain.

**When you edit Code.gs later:** you need to create a **new deployment
version** (Deploy > Manage deployments > pencil icon > New version) or the
live Web App URL keeps serving the old code.

## 5. Connect the HTML form to your deployment

1. Open **recovery-engagement-form.html** in a text editor.
2. Find this line near the top of the `<script>` section:
   ```js
   const SCRIPT_URL = "PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE";
   ```
3. Replace the placeholder with the Web App URL you copied in step 4, e.g.:
   ```js
   const SCRIPT_URL = "https://script.google.com/macros/s/XXXXXXXXXXXX/exec";
   ```
4. Save the file.

## 6. Use it

- Open `recovery-engagement-form.html` directly in a browser (double-click
  it) to test — no server needed, since it talks directly to your Apps
  Script Web App.
- To share it with your recovery agents, host it wherever you host your
  other tools (e.g. GitHub Pages, like your other CredPal projects), or
  just send them the HTML file.
- Dropdown options refresh from the **Lists** tab every time the form is
  loaded, so update option lists there and agents get the change next time
  they open the form.
- Click **Recent Submissions** at the bottom of the form to see the last 15
  entries pulled live from the Sheet.

## Field reference

| Form field | Type | Notes |
|---|---|---|
| Recovery Agent | Dropdown (required) | From Lists tab |
| Account ID | Text (required) | Free text |
| Engagement Channel | Dropdown (required) | From Lists tab |
| Engagement Status | Dropdown (required) | From Lists tab |
| Response | Dropdown | From Lists tab |
| Engagement Time | Time | |
| Engagement Date | Date (required) | |
| Payment Reported | Dropdown | Yes / No / Partial by default |
| Plan Type | Dropdown | From Lists tab |
| Plan Amount | Number | |
| Down-payment | Number | |
| Down-payment Date | Date | |
| Next Payment Date | Date | |
| Next Follow-up Date | Date | |
| Contact or Visit Notes | Text area | |
| Evidence Link/Reference | Text | e.g. Drive link |
| Changes Current Arrangement | Dropdown | Yes / No |

A `Timestamp` column is added automatically to every Entries row and isn't
part of the form.

## Known limitations (MVP)

- No login/authentication on the form — anyone with the link can submit.
  Add agent-side auth later if needed (the Credit Referral Platform build
  has a working email+password pattern you could reuse).
- The Sheet's own dropdown validation on the Entries tab covers the first
  2,000 rows; if you ever pass that, re-run `setupSheet` to extend it.
- List columns in the Lists tab watch the first 500 rows for options —
  plenty of headroom, but re-run `setupSheet` if you ever need more.
