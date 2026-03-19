# 🏡 宿泊税かんたん申告ツール — Japan Stay Tax Helper

> **⚠️ 免責事項 / Disclaimer**
> This is an **unofficial community tool**, not affiliated with any government authority or tax office. Always verify your final filing with official guidance from your prefecture or municipality.
> 本ツールは民間有志による非公式の申告補助ツールです。税務当局・地方自治体とは一切関係ありません。

---

## What is this?

A free, browser-based tool that helps accommodation operators in Japan calculate and generate their **宿泊税 (Accommodation / Stay Tax)** filing documents.

Upload a spreadsheet of your guest stays, select your region, and the tool auto-generates:

- **月計表（様式第8号）** — the official monthly stay-count report
- **宿泊税 納入申告書** — the declaration form for your tax authority

No server. No login. No data ever leaves your browser. Everything runs 100% client-side.

---

## ✨ Features

| Feature | Details |
|---|---|
| 🗾 **Multi-region support** | 8 prefectures/cities with built-in up-to-date tax rate tables |
| 📊 **Excel / CSV upload** | Smart column auto-detection — works with most PMS exports |
| 🗓️ **Auto month detection** | Filing periods are derived directly from your data — no manual selection |
| 📄 **Document generation** | Produces print-ready 月計表 and 納入申告書 in-browser |
| 📥 **Excel export** | Download results as a formatted .xlsx file |
| 💾 **Facility profiles** | Save and reload your facility details locally |
| 🌐 **4 languages** | Japanese · Chinese (Traditional) · English · Korean |
| 🖨️ **Print-ready** | Styled print stylesheet hides UI chrome for clean A4 output |

---

## 🗾 Supported Regions

| Region | Tax Name | Structure | Exempt Below |
|---|---|---|---|
| 東京都 | 東京都宿泊税 | ¥100 / ¥200 / ¥300 per person/night | None |
| 大阪府 | 大阪府宿泊税 | ¥200 / ¥400 / ¥500 per person/night | ¥5,000/pn |
| 京都市 | 京都市宿泊税 | ¥200 / ¥500 / ¥1,000 per person/night | None |
| 福岡県 | 福岡県宿泊税 | ¥200 / ¥500 per person/night | None |
| 北海道 倶知安町 | 倶知安町宿泊税 | 2% of fare | None |
| 石川県 金沢市 | 金沢市宿泊税 | ¥200 / ¥500 per person/night | None |
| 長野県 白馬村・小谷村 | 長野県宿泊税 | ¥150 / ¥300 per person/night | None |
| 奈良県 | 奈良県宿泊税 | ¥200 / ¥500 per person/night | None |

Tax rates are based on the **per-person, per-night fare** (宿泊料金 ÷ 泊数 ÷ 人数).

---

## 🚀 How to use

### Option A — Open directly in your browser

Just open `japan_stay_tax_portal_v0.2.html` in any modern browser. No installation, no build step, no internet connection required after the first load (fonts are fetched from Google Fonts).

### Option B — Host it yourself

Drop the single HTML file onto any static hosting (GitHub Pages, Netlify, S3, etc.) and share the URL with your staff.

```
# Example: serve locally with Python
python3 -m http.server 8080
# then open http://localhost:8080/japan_stay_tax_portal_v0.2.html
```

---

## 📋 How to prepare your data

The tool accepts `.xlsx`, `.xls`, and `.csv` files. Column order is auto-detected from the header row — it understands Japanese, English, Chinese, and Korean header names.

### Expected columns

| Column | Content | Notes |
|---|---|---|
| **A — Check-in date** | `YYYY/MM/DD` | Excel date cells also work |
| **B — Nights** | Integer | e.g. `2` |
| **C — Guests** | Integer | e.g. `3` |
| **D — Total fare** | Number (¥) | Total for the entire stay, tax-excluded. No breakfast / cleaning fees. |
| **E — Reservation ID** | Any | Optional. Used for your reference only. |

> **Important:** Column D must be the **total stay fare** (`nights × nightly rate × rooms`), not a per-night amount.
> The tool divides this by nights and guests to derive the per-person-per-night rate used for tax bracket classification.

Download the built-in Excel template from **Step 2** in the tool for a ready-to-fill starting point.

---

## 🛠️ How it works

```
Upload Excel / CSV
       │
       ▼
Smart header detection  ──►  Column mapping (date / nights / guests / fare / ID)
       │
       ▼
Per-record calculation
  • ppn = fare ÷ nights ÷ guests          (per-person-per-night price)
  • tax bracket = region rate table lookup
  • tax = rate × nights × guests           (or fare × 2% for Kutchan)
       │
       ▼
Aggregate by calendar month  ──►  Auto-detect all year/month combinations in data
       │
       ├──►  月計表 (monthly stay-count table, 様式第8号)
       └──►  納入申告書 (tax declaration form)
```

All computation runs in the browser with zero external API calls.

---

## 🐛 Known bugs fixed (v0.1 → v0.2)

### v0.1 bug — Nights column misread as fare amount
**Root cause:** The header-detection regex `泊数?` matched `宿泊料金合計` (the fare column header) because `宿` + `泊` contains `泊`. This caused `niC` (nights column index) to be overwritten with the fare column index, so `nights` was being read as a ¥ figure.

**Fixes applied in v0.2:**
1. Regex changed to `(?<!宿)泊数?` — negative lookbehind excludes `宿泊` prefix
2. Post-detection guard: `if(niC === faC){ niC = 1; }` resets to default column B if collision is detected
3. `nights` capped at 365 as a last-resort sanity check

### v0.2 — Filing year and month inputs removed
Previously, users had to manually select the filing year and tick month checkboxes. In v0.2, **filing periods are derived automatically** from the check-in dates in the uploaded data. If your file covers March, August, and September, those three month-tables are generated without any manual configuration.

---

## 📁 File structure

```
japan_stay_tax_portal_v0.2.html   ← The entire app. Single file, no dependencies.
README.md
```

Everything — HTML, CSS, JavaScript, the embedded Excel template, and the regional tax rate database — lives in one self-contained HTML file. There is no build process and no package.json.

The only external resource loaded at runtime is the **SheetJS (xlsx) library** from cdnjs and **Noto fonts** from Google Fonts. The app will still function (with system fonts) if those are unavailable.

---

## 🌐 Internationalisation

The UI is fully translated in 4 languages, switchable from the header at any time:

- 🇯🇵 **日本語** (Japanese) — default
- 🇨🇳 **中文** (Traditional Chinese)
- 🇬🇧 **English**
- 🇰🇷 **한국어** (Korean)

All translation strings live in the `T` object near the top of the `<script>` block. Adding a new language is a matter of adding a new key to `T` and a button to the header `lang-row` div.

---

## 🔒 Privacy

- **No data is ever uploaded to a server.** File parsing, tax calculation, and document generation all happen in your browser using client-side JavaScript.
- Facility profiles are saved to `localStorage` — local to your device and browser only.
- The tool makes no network requests except loading fonts and the SheetJS library from CDN on first load.

---

## 🤝 Contributing

Pull requests are welcome. Common areas for contribution:

- **Adding regions** — Add a new entry to the `REGIONS` object following the existing pattern. Fixed-rate regions use the bracket array; percentage-rate regions (like Kutchan) set `is_pct_tax: true`.
- **Updating tax rates** — Rates change occasionally. Each region entry has clearly named `rates`, `exempt_below`, and `since` fields.
- **Adding languages** — Add a translation object to `T` and a `<button>` to the `.lang-row` in the header.
- **Bug reports** — Please include your browser, a description of the Excel file structure (headers, date format), and the region selected.

### Adding a new region

```javascript
// In the REGIONS object:
your_region_key: {
  name_ja:'地域名', name_zh:'地区名', name_en:'Region Name', name_ko:'지역명',
  tax_name:'〇〇宿泊税',
  rates:[
    {label:'①', min:0,    max:10000, rate:200, color:'#e8f5e9', tc:'#1b5e20'},
    {label:'②', min:10000, max:null, rate:500, color:'#fce4ec', tc:'#880e4f'},
  ],
  exempt_below: null,    // or a number, e.g. 5000
  office:{ ja:'担当窓口', zh:'…', en:'…', ko:'…' },
  address:'〒xxx-xxxx …',
  tel:'0X-XXXX-XXXX',
  url:'https://…',
  since:'20XX年X月',
  note_ja:'備考があれば',
},
```

---

## 📜 Changelog

### v0.2 (2025)
- **Removed** manual filing year input and month-selection checkbox grid
- Filing periods now **auto-detected** from uploaded data — supports multi-year data spanning two calendar years correctly
- Profile save/load no longer stores or restores year/month values
- `exportProfile` / `importProfile` Excel functions updated to match

### v0.1 (2025)
- **Fixed** critical bug: nights column regex `泊数?` incorrectly matched `宿泊料金合計`, causing fare value to be read as number of nights
- Regex updated to `(?<!宿)泊数?` (negative lookbehind)
- Added post-detection guard `if(niC === faC){ niC = 1; }`
- Added `nights` cap at 365

### v0 (Initial release)
- Multi-region tax rate database (8 regions)
- Excel/CSV upload with smart header detection
- 月計表 and 納入申告書 generation
- 4-language UI
- Facility profile save/load
- Excel result export

---

## ⚖️ Legal

This tool is an **unofficial community project**. It is not affiliated with, endorsed by, or connected to any Japanese government authority, tax office, or prefecture.

- The creators accept no responsibility for errors in tax filings or any resulting damages
- Tax rates and regulations change — always verify with your official prefectural tax authority before submitting
- This tool is distributed freely. You may modify and redistribute it, but **do not represent it as an official government tool**

---

*Maintained by community volunteers. If this tool saved you time, consider contributing a bug report, a rate update, or a new region.*
