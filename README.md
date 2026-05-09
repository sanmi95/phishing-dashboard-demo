# Phishing Dashboard - Campaign Progress Tracker

> A **browser-based, zero-backend phishing simulation analytics dashboard** built for security awareness teams. Upload CSV exports from any phishing simulation platform, track trends across campaigns, and generate actionable insights - all processed locally with no data ever leaving the browser.

---

## Overview

The Phishing Dashboard gives security awareness teams a clean, branded analytics hub to track phishing simulation performance over time. It features a bold candy-stripe color palette and a fully responsive single-file architecture.

**Key characteristics:**
- Single HTML file with no framework, no server, and no dependencies to install
- 100% client-side processing (CSV parsed in-browser via PapaParse)
- Multi-campaign tracking with trend analysis across time
- Department-level and individual-level (repeat offender) intelligence
- Chart.js-powered visualizations with 7 distinct chart types
- Deployable to any static host (Netlify, GitHub Pages, etc.)

---

## How It Works

The diagram below shows the complete flow of the application - from page load through user interactions, CSV upload, and re-rendering.

<p align="center">
  <img src="docs/app-flowchart.svg" alt="Phishing Dashboard app flowchart" width="600"/>
</p>

**Key points to understand from the diagram:**

- On load, the app seeds 3 demo campaigns so the UI is never empty
- `render()` is the single dispatcher; everything re-renders from it
- The two tabs (Dashboard and Manage) share the same state array; switching tabs just calls `render()` again
- CSV parsing happens entirely in PapaParse (browser-only); the result is pushed to the in-memory state array and `render()` is called again
- Deleting a campaign removes it from the state array and re-renders with no persistence layer involved

---

## Metrics Tracked

The dashboard computes and visualizes the following metrics per campaign:

| Metric | Description |
|--------|-------------|
| **Failure Rate** | % of delivered users who submitted credentials |
| **Click Rate** | % of delivered users who clicked the phishing link |
| **Cred Rate** | % of delivered users who supplied credentials |
| **Report Rate** | % of users who reported the phishing email |
| **Resiliency Score** | Composite: `((reported + did_not_click) / delivered) × 100` |
| **Training Completion** | % of total recipients who completed remedial training |
| **Compromised Count** | Raw count of fully compromised accounts |
| **Reported Count** | Raw count of users who correctly reported |

All metrics display a **delta badge** comparing the current campaign to the previous one, with directional color coding (green = improvement, red = regression).

---

## UI Components

### Dashboard Tab
- **Campaign Strip** - scrollable horizontal chips, one per campaign plus an "All" aggregate view
- **KPI Row** - 8 metric cards with delta badges (color-coded vs. previous campaign)
- **Trend Analysis Section** - 5 Chart.js charts (line and bar)
- **Department Intelligence** - grouped bar chart plus sortable inline bar rows with delta
- **Repeat Offenders** - cross-campaign recidivism tracking with name, dept, country

### Manage Campaigns Tab
- **Tabular campaign list** - all campaigns with failure rate pill, resiliency pill, trend arrow
- **View / Delete** actions per row
- **Upload Campaign** modal - drag-and-drop or browse, campaign name input, CSV parse

### Upload Modal
- Drag-and-drop zone with visual feedback on hover/drag-over
- File name confirmation
- Campaign name text input
- All data parsed locally before any state mutation

---

## Getting Started

### Option 1 - Open directly in browser

```bash
# No installation required
open index.html
# or double-click the file in your file explorer
```

### Option 2 - Serve locally

```bash
# Python 3
python -m http.server 8080

# Node (npx)
npx serve .

# Then open: http://localhost:8080
```

### Option 3 - Deploy to Netlify

```bash
# Drag the folder to app.netlify.com/drop
# or use the Netlify CLI:

npm install -g netlify-cli
netlify deploy --prod --dir .
```

### Option 4 - Deploy to GitHub Pages

1. Push `index.html` to the root of a public GitHub repo
2. Go to **Settings > Pages**
3. Set source to `main` branch, root folder
4. Your dashboard is live at `https://<username>.github.io/<repo>`

---

## CSV Format

The dashboard auto-detects columns from your phishing platform's CSV export. The following column names are expected (GoPhish / KnowBe4 style):

```csv
FirstName,LastName,Email,Department,Position,Compromised,
SuccessfullyDeliveredEmail_TimeStamp,EmailLinkClicked_TimeStamp,
CredSupplied_TimeStamp(Compromised),Phishing Reported On,
Training Status,Country
```

**Minimum required columns** for basic metrics:
- `Compromised` - boolean (`true` / `false`)
- `Department` - string

**Optional columns** (enable additional metrics when present):
- `SuccessfullyDeliveredEmail_TimeStamp` - enables delivery rate
- `EmailLinkClicked_TimeStamp` - enables click rate
- `CredSupplied_TimeStamp(Compromised)` - enables cred supply rate
- `Phishing Reported On` - enables report rate
- `Training Status` - enables training completion rate (looks for "complet" substring)

> Columns not present in your export are silently skipped; affected metrics will display as 0.

---

## Privacy & Data Handling

| Concern | How it's handled |
|---------|-----------------|
| **PII in CSVs** | Never transmitted; all parsing is local (PapaParse, in-browser) |
| **Session persistence** | Data lives in JS memory only; cleared on page refresh |
| **Network requests** | Only CDN requests for Chart.js, PapaParse, and Google Fonts |
| **Server** | None; this is a static single-file application |
| **Credentials** | No authentication layer in base version |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | Vanilla HTML5 |
| Styling | Vanilla CSS3 (CSS custom properties, Grid, Flexbox) |
| Logic | Vanilla JavaScript (ES6+, no build step) |
| Charts | [Chart.js 4.4.1](https://www.chartjs.org/) via CDN |
| CSV Parsing | [PapaParse 5.4.1](https://www.papaparse.com/) via CDN |
| Fonts | Google Fonts: Nunito (UI) and Fira Code (monospace metrics) |
| Hosting (optional) | Netlify / GitHub Pages / any static host |

---

## Design System

The dashboard uses a bold candy-stripe palette built from CSS custom properties:

| Token | Hex | Usage |
|-------|-----|-------|
| `--red` | `#d62b2b` | Failure metrics, danger states, primary CTA |
| `--yellow` | `#f5c400` | Click metrics, warning states, modal accent |
| `--blue` | `#1a4fa0` | Active nav, resiliency, primary info |
| `--green` | `#2e8b3a` | Report rate, training, positive delta |
| `--orange` | `#e8671a` | Cred rate, secondary warning |
| `--brown` | `#6b3a2a` | Accent (dept chart line) |
| `--bg` | `#faf8f5` | Page background |
| `--surface` | `#ffffff` | Card / modal background |
| `--ink` | `#1a1a2e` | Primary text |
| `--muted` | `#7a7068` | Labels, secondary text |

The **candy-stripe top bar** cycles through all palette colors as a signature visual identity element.

---

## 🧪 Demo Data

The dashboard ships with 3 seeded campaigns so it renders immediately without uploading a file:

| Campaign | Date | Failure Rate | Resiliency |
|----------|------|-------------|------------|
| Q3 2024 - Invoice Phishing | Aug 12, 2024 | 28.5% | 42% |
| Q4 2024 - HR Portal Reset | Nov 5, 2024 | 21.9% | 55% |
| Q1 2025 - Package Delivery | Feb 18, 2025 | 16.1% | 67% |

The demo data shows a clear downward trend in failure rate and upward trend in resiliency, illustrating the expected program maturity curve.

---

## Project Structure

```
phishing-dashboard/
├── index.html              # Entire application - HTML + CSS + JS in one file
│   ├── <style>             # All styling (CSS custom properties, layout, components)
│   ├── <body>              # Markup (header, modal, toast, main container)
│   └── <script>
│       ├── SEED[]          # Demo campaign data
│       ├── OFFENDERS[]     # Demo repeat offender data
│       ├── render()        # Main render dispatcher
│       ├── dashHtml()      # Dashboard tab HTML generator
│       ├── renderManage()  # Manage tab renderer
│       ├── uploadCSV()     # CSV parse + campaign creation
│       ├── build*()        # Chart.js chart builders (6 functions)
│       └── renderDepts()   # Department row renderer
└── docs/
    └── app-flowchart.svg   # App flowchart (referenced in this README)
```

---

## Author

**Gerardo Hernandez**
Security Analyst | Adjunct Professor | Doctoral Candidate in Engineering (AI/ML)

---

## License

MIT License - feel free to fork and adapt for your own security awareness program.

---

## Acknowledgments

- [Chart.js](https://www.chartjs.org/) - open-source chart library
- [PapaParse](https://www.papaparse.com/) - browser CSV parser
- [Nunito & Fira Code](https://fonts.google.com/) - Google Fonts
