# ฿xpense — Personal Finance Dashboard

A fully free, automated personal finance dashboard for tracking expenses in Thai Baht. Screenshots of receipts are automatically processed by AI and displayed in a mobile-friendly web app hosted on GitHub Pages.

**Live dashboard:** https://xenirio.github.io/finance-tracker

---

## How It Works

```
Receipt photo → Dropbox /Expenses/
    → Google Apps Script (runs every 1 min)
        → Gemini 2.5 Flash (extracts merchant, amount, date, category)
            → Google Sheets (stores data)
                → Dashboard (reads & displays)
```

---

## Stack

| Part | Tool | Cost |
|---|---|---|
| Receipt storage | Dropbox | Free (2GB) |
| Automation | Google Apps Script | Free |
| AI extraction | Gemini 2.5 Flash | Free (AI Studio) |
| Data storage | Google Sheets | Free |
| Dashboard hosting | GitHub Pages | Free |

---

## Setup Guide

### 1. Google Sheet

1. Create a new Google Sheet named `finance-tracker`
2. Rename the first tab to `Transactions`
3. Add these headers in row 1:

| A | B | C | D |
|---|---|---|---|
| Date | Merchant | Amount | Category |

4. Share the sheet: **Share → Anyone with the link → Viewer**
5. Copy the Sheet ID from the URL — the long string between `/d/` and `/edit`

---

### 2. Dropbox App

1. Go to [dropbox.com/developers/apps](https://www.dropbox.com/developers/apps)
2. Click **Create app** → **Scoped access** → **Full Dropbox**
3. Name it `expense-reader` → **Create app**
4. Under **Permissions** tab, enable:
   - ✅ `files.content.read`
   - ✅ `files.content.write`
   - ✅ `files.metadata.read`
5. Click **Submit**
6. Back on **Settings** tab, note your **App key** and **App secret**

**Get a refresh token (never expires):**

In your browser, open this URL (replace `YOUR_APP_KEY`):
```
https://www.dropbox.com/oauth2/authorize?client_id=YOUR_APP_KEY&response_type=code&token_access_type=offline
```
Click **Allow** and copy the authorization code shown.

Then run `getRefreshToken` in Apps Script (see step 3) to exchange it for a refresh token.

---

### 3. Google Apps Script

1. Open your Google Sheet → **Extensions → Apps Script**
2. Paste the full script from `Code.gs` in this repo
3. Fill in the CONFIG section at the top:

```javascript
const APP_KEY        = 'your_dropbox_app_key';
const APP_SECRET     = 'your_dropbox_app_secret';
const REFRESH_TOKEN  = 'your_dropbox_refresh_token';
const DROPBOX_FOLDER = '/Expenses';
const GEMINI_API_KEY = 'your_gemini_api_key';
const SHEET_NAME     = 'Transactions';
```

4. Run `getRefreshToken` once to get your Dropbox refresh token — copy it from the logs
5. Paste the refresh token into `REFRESH_TOKEN`
6. Run `createTrigger` once to start automation:
   - Checks for new receipts every **1 minute**
   - Cleans up old files daily at **3am**

**Get a free Gemini API key:** [aistudio.google.com](https://aistudio.google.com) → Get API key

---

### 4. Dashboard

1. Open `index.html` and set your Sheet ID:

```javascript
const SHEET_ID = 'your_google_sheet_id_here';
```

2. Create a GitHub repo named `finance-tracker`
3. Upload `index.html` to the root
4. Go to **Settings → Pages → Branch: main / root → Save**
5. Dashboard is live at `https://YOUR_USERNAME.github.io/finance-tracker`

**Add to iPhone home screen:** Safari → Share → Add to Home Screen

---

## Adding Receipts

**Option A — Share Sheet (recommended)**
1. Take a screenshot of any receipt, bank statement, or transaction list
2. Tap **Share → Dropbox → Expenses folder**
3. Apps Script picks it up within 1 minute

**Option B — Dropbox app camera scan**
1. Open Dropbox → tap **+** → **Scan document**
2. Scan receipt → save to `/Expenses/`

Gemini automatically:
- Detects if the image is a receipt (non-receipts are deleted)
- Extracts all transactions from a single screenshot (supports multi-row bank statements)
- Cleans up merchant names
- Handles Thai date formats and Buddhist Era years

---

## Categories

| Category | Examples |
|---|---|
| Food & Dining | Restaurants, cafes, delivery, supermarkets |
| Transport | Grab, BTS, fuel, taxis |
| Bills & Utilities | AIS, True Move, electricity, internet |
| Subscriptions | Netflix, Spotify, iCloud, ChatGPT |
| Shopping | Lazada, Shopee, Central, clothing |
| Health & Pharmacy | Boots, Watsons, hospitals, clinics |
| Education | Udemy, Coursera, books, courses |
| Entertainment | Cinema, concerts, games |
| Travel | Hotels, flights, Airbnb, Klook |
| Other | ATM, transfers, miscellaneous |

---

## Dashboard Features

- **Daily view** — bar chart of every day in the month, tap to select a day
- **Monthly view** — full month totals with 6-month trend per category
- **Yearly view** — 12-month breakdown, tap any month to drill down
- Category breakdown with animated color bar
- Per-category sparkline trend tiles
- Recent transactions list
- Mobile-first, works as iPhone home screen app

---

## File Structure

```
finance-tracker/
├── index.html      # Dashboard (GitHub Pages)
├── Code.gs         # Google Apps Script
└── README.md
```

---

## Automation Details

| Trigger | Function | Schedule |
|---|---|---|
| `processNewReceipts` | Scan Dropbox, call Gemini, write to Sheets | Every 1 minute |
| `deleteOldFiles` | Remove files older than 7 days from Dropbox | Daily at 3am |

Duplicate protection: processed file IDs are stored in a hidden `_Processed` tab in the Google Sheet. Each file is only processed once regardless of how many times the trigger runs.
