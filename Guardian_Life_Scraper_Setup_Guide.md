# 🚀 Guardian Life Scraper — Full Setup Guide
**Auto-runs every Monday, Wednesday & Friday at 8:00 AM IST**

---

## 📋 What You Need Before Starting
- A **GitHub account** (free) → [github.com](https://github.com)
- Your 5 project files:
  - `guardian_life_scraper_github.py`
  - `requirements.txt`
  - `scraper.yml`
  - `README.md`
  - `.github` (folder — used for workflow)

---

## STEP 1 — Create a New GitHub Repository

1. Go to [github.com](https://github.com) and **log in**
2. Click the **➕ New** button (top right) → **New repository**
3. Fill in:
   - **Repository name:** `guardian-life-scraper`
   - **Visibility:** ✅ Private (recommended)
   - **Do NOT** check "Add a README" (you already have one)
4. Click **Create repository**

---

## STEP 2 — Upload Your Files

### Option A — Upload via GitHub Website (Easiest)

1. On your new repo page, click **"uploading an existing file"** link
2. Drag and drop these files:
   - `guardian_life_scraper_github.py`
   - `requirements.txt`
   - `README.md`
3. Click **Commit changes**

### Create the Workflow Folder (Important!)

The `scraper.yml` file must go inside a special folder path: `.github/workflows/`

1. On your repo page, click **Add file → Create new file**
2. In the filename box, type exactly: `.github/workflows/scraper.yml`
   - GitHub will auto-create the folders as you type the `/` slashes
3. Paste the contents of your `scraper.yml` file (see updated content below)
4. Click **Commit new file**

---

## STEP 3 — Update the Schedule to 8:00 AM IST

> **8:00 AM IST = 2:30 AM UTC**
> Your current file has `30 3 * * 1,3,5` which is **9:00 AM IST**.
> Change it to `30 2 * * 1,3,5` for **8:00 AM IST**.

Use this updated `scraper.yml` content:

```yaml
name: Guardian Life Career Scraper - Auto Schedule

on:
  schedule:
    # Runs at 2:30 AM UTC = 8:00 AM IST (Mon, Wed, Fri)
    - cron: '30 2 * * 1,3,5'
  
  # Allow manual trigger from GitHub UI
  workflow_dispatch:

jobs:
  scrape-jobs:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install openpyxl beautifulsoup4 requests pandas tqdm
      
      - name: Run scraper
        run: |
          python guardian_life_scraper_github.py
      
      - name: Upload Excel file
        uses: actions/upload-artifact@v3
        with:
          name: scraped-jobs-excel
          path: output/*.xlsx
          retention-days: 90
      
      - name: Upload CSV file
        uses: actions/upload-artifact@v3
        with:
          name: scraped-jobs-csv
          path: output/*.csv
          retention-days: 90
      
      - name: Upload logs
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: scraper-logs
          path: logs/*.log
          retention-days: 30
      
      - name: Commit and push results to repository
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "GitHub Actions Bot"
          git add output/*.xlsx output/*.csv logs/*.json || true
          git commit -m "Auto: Scraped jobs on $(date +'%Y-%m-%d')" || true
          git push || true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## STEP 4 — Test the Scraper Manually (First Run)

Before waiting for the schedule, test it now:

1. Go to your repo → click the **"Actions"** tab
2. Click **"Guardian Life Career Scraper - Auto Schedule"** in the left list
3. Click **"Run workflow"** button (right side) → **"Run workflow"**
4. Wait 2–3 minutes and refresh the page
5. Click on the workflow run to see results

### ✅ Success looks like:
- Green checkmark ✅ next to the run
- Output files visible under **"Artifacts"** section

### ❌ If it fails:
- Click the failed step to read the error log
- Most common issue: cookies expired (see Troubleshooting below)

---

## STEP 5 — Download Your Files After Each Run

After every auto-run:

1. Go to **Actions tab** → click the latest run
2. Scroll down to **"Artifacts"** section
3. Click **"scraped-jobs-excel"** to download the `.xlsx` file
4. Click **"scraped-jobs-csv"** to download the `.csv` file

> **Note:** Artifacts are stored for **90 days** automatically.

Alternatively, the files are also **committed directly to your repository** in the `output/` folder — you can browse them there anytime.

---

## 📅 Your Auto-Schedule Summary

| Day | Time (IST) | Time (UTC) |
|-----|-----------|-----------|
| Monday | 8:00 AM | 2:30 AM |
| Wednesday | 8:00 AM | 2:30 AM |
| Friday | 8:00 AM | 2:30 AM |

---

## ⚠️ Troubleshooting

### Problem: Scraper runs but finds 0 jobs / authentication error
**Cause:** The cookies in your Python file have expired.

**Fix:**
1. Open Guardian Life careers in Chrome: `https://guardianlife.wd5.myworkdayjobs.com`
2. Press **F12** → go to **Network** tab
3. Apply for any filter, find a `/jobs` API request
4. Right-click → **Copy → Copy as cURL**
5. Extract the new cookie values from the copied text
6. Update the `cookies` dictionary in `guardian_life_scraper_github.py`
7. Push the updated file to GitHub

### Problem: GitHub Actions not running on schedule
**Cause:** GitHub sometimes delays or skips scheduled runs on inactive repos.

**Fix:** Make a small commit (edit README) to keep the repo "active". Alternatively, manually trigger the run from the Actions tab.

### Problem: "Permission denied" when pushing results
**Fix:** Go to repo **Settings → Actions → General → Workflow permissions** → select **"Read and write permissions"** → Save.

---

## 🔔 Optional: Get Email Notifications on Failure

1. Go to GitHub → **Profile (top right) → Settings**
2. Click **Notifications**
3. Under **"GitHub Actions"**, enable **"Send notifications for failed workflows only"**
4. You'll get an email if the scraper ever fails

---

## 📁 Final Folder Structure in Your Repo

```
guardian-life-scraper/
├── .github/
│   └── workflows/
│       └── scraper.yml          ← Schedule & automation config
├── output/
│   ├── GuardianLife_Jobs_2025-01-20.xlsx
│   ├── GuardianLife_Jobs_2025-01-20.csv
│   └── GuardianLife_Jobs_2025-01-20.json
├── logs/
│   ├── scraper_2025-01-20_08-30-00.log
│   └── run_history.json
├── guardian_life_scraper_github.py  ← Main scraper
├── requirements.txt
└── README.md
```

---

## ✅ Quick Checklist

- [ ] GitHub account created
- [ ] New private repository created: `guardian-life-scraper`
- [ ] `guardian_life_scraper_github.py` uploaded
- [ ] `requirements.txt` uploaded
- [ ] `README.md` uploaded
- [ ] `.github/workflows/scraper.yml` created with updated cron `30 2 * * 1,3,5`
- [ ] Repo Settings → Actions → Workflow permissions set to **Read and write**
- [ ] Manual test run completed successfully ✅
- [ ] Email notifications enabled for failures

---

*Setup complete! Your scraper will now automatically run every Monday, Wednesday, and Friday at 8:00 AM IST and save the output files to your repository.*
