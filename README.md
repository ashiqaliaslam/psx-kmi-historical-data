# PSX KMI All-Share Islamic Index — Historical Shariah Compliance Dataset

A comprehensive, open-source historical archive of semi-annual **PSX–Meezan KMI All-Share Islamic Index** re-composition reviews, complete with financial screening ratios, Net Liquid Assets (NLA), Shariah Star Ratings, and compliance status across multiple years.

Updated after every official semi-annual PSX re-composition notice. No API key, no signup — fetch directly via raw GitHub URLs in JSON or CSV format.

## Available Files & Endpoints

Replace `<username>/<repo>` with your GitHub repository path:

| Path | Format | Description |
| ----- | ----- | ----- |
| `data/kmi_all_reviews.csv` | CSV | **Master Historical Archive** containing all semi-annual KMI All-Share reviews stacked chronologically. |
| `data/kmi_all_reviews.json` | JSON | Auto-generated full historical JSON array (ideal for point-in-time historical dividend purification / *Tazkiyah* and multi-year ratio tracking). |
| `data/kmi_latest.json` | JSON | Auto-generated deduplicated snapshot containing **only the latest `effective_date` record per ticker** for active Shariah screening. |

## Dataset Schema (`15 Columns`)

All column headers follow strict lowercase `snake_case` naming for seamless consumption across REST APIs, JavaScript/React apps, Python/Pandas, and SQL databases:

| Column | Type | Example | Description |
| ----- | ----- | ----- | ----- |
| `effective_date` | `DD-MM-YYYY` | `30-11-2026` | Date the re-composition list takes effect on PSX. |
| `review_period` | `DD-MM-YYYY` | `30-06-2026` | Underlying financial statement cutoff date audited for the review. |
| `sr_no` | `integer` | `1` | Serial number from the official PSX notice. |
| `ticker` | `string` | `MEBL` | PSX stock symbol / ticker. |
| `company_name` | `string` | `Meezan Bank Limited` | Registered company name on PSX. |
| `objective` | `string` | `Pass` | Core business activity screening result (`Pass` / `Fail`). |
| `debt_ratio` | `float` | `14.50` | Interest-Bearing Debt to Total Assets (`%`). |
| `investment_ratio` | `float` | `9.80` | Non-Compliant Investments to Total Assets (`%`). |
| `income_ratio` | `float` | `1.65` | Non-Compliant Income to Total Revenue (`%` — used for Dividend Purification / *Tazkiyah*). |
| `illiquid_assets_ratio` | `float` | `68.00` | Illiquid Assets to Total Assets (`%`). |
| `nla_per_share_ratio` | `float` | `210.50` | Net Liquid Assets per share (PKR). |
| `share_price_as_of_review_period` | `float` | `945.00` | Closing share price on the financial period cutoff date (PKR). |
| `shariah_status` | `string` | `Compliant` | Final Shariah status (`Compliant` / `Non-Compliant`). |
| `star_rating` | `integer` | `4` | Official Shariah Compliance Star Rating (`3`, `4`, or `5` stars; blank on pre-2026 reviews). |
| `footnotes` | `string` | `Note 1` | Official remarks or footnote references from PSX notices. |

## PSX–Meezan Shariah Screening Criteria

A company must satisfy **all six** screening rules to be classified as Shariah Compliant:

1. **Core Business Activity (`objective`):** Halal primary business (excludes conventional banking, conventional insurance, alcohol, gambling, pork, tobacco, and explosives).

2. **Interest-Bearing Debt (`debt_ratio`):** Must be **`< 33%`** of Total Assets *(reduced from 37% effective November 2026 re-composition)*.

3. **Non-Compliant Investments (`investment_ratio`):** Must be **`< 33%`** of Total Assets.

4. **Non-Compliant Income (`income_ratio`):** Must be **`< 5%`** of Total Revenue.

5. **Illiquid Assets (`illiquid_assets_ratio`):** Must be **`>= 25%`** of Total Assets.

6. **Net Liquid Assets vs. Share Price (`nla_per_share_ratio`):** Market price per share must be **`>= NLA per share`**.

### Shariah Compliance Star Ratings (Introduced 2026)

* **5 Stars (`5★`):** `debt_ratio = 0%`, `investment_ratio = 0%`, and `income_ratio = 0%`

* **4 Stars (`4★`):** `debt_ratio < 11%`, `investment_ratio < 11%`, and `income_ratio < 2%`

* **3 Stars (`3★`):** `debt_ratio < 22%`, `investment_ratio < 22%`, and `income_ratio < 4%`

## Quick Usage Examples

### Fetch Latest Active KMI List (JavaScript)

```javascript
const res = await fetch(
  'https://raw.githubusercontent.com/<username>/<repo>/main/data/kmi_latest.json'
);
const latestKmiList = await res.json();
const mebl = latestKmiList.find((item) => item.ticker === 'MEBL');
console.log(mebl.shariah_status, mebl.star_rating, mebl.income_ratio);
```

### Point-in-Time Dividend Purification Lookup (JavaScript)

Convert `DD-MM-YYYY` dates to `YYYY-MM-DD` before comparing against a dividend's `ex_date`:

```javascript
function dmyToIso(dmy) {
  const [dd, mm, yyyy] = String(dmy || '').split('-');
  return yyyy && mm && dd ? `${yyyy}-${mm.padStart(2, '0')}-${dd.padStart(2, '0')}` : '';
}

function getHistoricalPurificationPct(allReviews, ticker, exDateIso) {
  const matches = allReviews
    .filter((r) => r.ticker === ticker && dmyToIso(r.effective_date) <= exDateIso)
    .sort((a, b) => dmyToIso(b.effective_date).localeCompare(dmyToIso(a.effective_date)));

  return matches.length > 0 ? parseFloat(matches[0].income_ratio) || 0 : 0;
}
```

### Load Full Archive in Python (Pandas)

```python
import pandas as pd

url = "https://raw.githubusercontent.com/<username>/<repo>/main/data/kmi_all_reviews.csv"
df = pd.read_csv(url)
df["effective_date_dt"] = pd.to_datetime(df["effective_date"], format="%d-%m-%Y")
print(df.groupby("review_period")["ticker"].count())
```

## Disclaimer

Data is compiled from official **Pakistan Stock Exchange (PSX)** KMI All-Share Index re-composition notices for informational, research, and portfolio analytics purposes only. Always verify critical investment or *Tazkiyah* decisions against official PSX / Al Meezan notices.
