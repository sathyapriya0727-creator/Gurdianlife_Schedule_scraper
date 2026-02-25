# Guardian Life Career Scraper

Automated scraper for [Guardian Life job postings](https://guardianlife.wd5.myworkdayjobs.com/Guardian-Life-Careers).

Runs automatically every **Monday, Wednesday & Friday at 8:00 AM IST** via GitHub Actions.

## Output
- Excel (`.xlsx`) — formatted with headers, alternating rows, auto-filter
- CSV (`.csv`) — for data analysis
- JSON (`.json`) — for developers / history tracking

Scraped files are saved in the `output/` folder and also available as
downloadable Artifacts from the Actions tab.

## Schedule
`cron: '30 2 * * 1,3,5'` → 2:30 AM UTC = 8:00 AM IST, Mon/Wed/Fri

## Manual Run
Go to **Actions → Guardian Life Career Scraper → Run workflow**

## Cookie Refresh
If 0 jobs are returned, cookies in `guardian_life_scraper_github.py` have expired.
See the Troubleshooting section in `SETUP_GUIDE.md`.
