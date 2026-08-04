# CSV Dashboard Builder

A single-file, browser-based dashboard tool that ingests raw CSV data and lets you build interactive report visuals — bar, line, area, pie, donut, scatter, KPI cards, and tables — styled after Microsoft Power BI. No install, no server, no build step. Open the HTML file and go.

**Built by Nova Agentic**

---

## Table of contents

1. [What this tool does](#what-this-tool-does)
2. [How it's built](#how-its-built)
3. [Getting started](#getting-started)
4. [Step-by-step: using the tool](#step-by-step-using-the-tool)
5. [How the data pipeline works](#how-the-data-pipeline-works)
6. [How visuals are configured](#how-visuals-are-configured)
7. [Chart types explained](#chart-types-explained)
8. [Slicers (filtering)](#slicers-filtering)
9. [Color scheme](#color-scheme)
10. [File structure](#file-structure)
11. [Known limitations](#known-limitations)
12. [Browser support](#browser-support)
13. [License / credit](#license--credit)

---

## What this tool does

`csv-dashboard-builder.html` is a self-contained report canvas, similar in spirit to a Power BI report page. You:

1. Upload one or more raw CSV files.
2. The tool automatically detects each column's data type (number, date, or text).
3. You add "visual" cards to the canvas and, for each one, choose:
   - the chart type,
   - which column drives the axis/categories,
   - which column(s) supply the values,
   - how those values should be summarized (sum, average, count, min, max).
4. You can add slicers (dropdown filters) that filter every visual built from the same CSV.

Everything runs client-side in your browser. Your CSV data is never uploaded to a server — it's parsed and held in memory by the page itself.

---

## How it's built

The entire tool is one HTML file containing:

- **HTML** — the ribbon (top toolbar), the Fields pane (left), the report canvas (center), and the Visualization pane (right).
- **CSS** — a Power BI–style design system using CSS custom properties (variables) for color, so the light/dark canvas toggle works by swapping a single set of variables.
- **JavaScript** — all app logic: CSV parsing, type detection, state management, chart rendering, and the interactive configuration panels.

Two open-source libraries are loaded from a CDN at the top of the file:

| Library | Purpose |
|---|---|
| [PapaParse](https://www.papaparse.com/) | Parses uploaded CSV files into JavaScript arrays/objects |
| [Chart.js](https://www.chartjs.org/) | Renders bar, line, area, pie, donut, and scatter charts on HTML `<canvas>` elements |

Because it's a static file, it can be opened directly (double-click) or hosted anywhere — GitHub Pages, a static file server, or your local machine.

---

## Getting started

### Option A — Just open it
1. Download `csv-dashboard-builder.html`.
2. Double-click it (or right-click → Open With → your browser).
3. It opens as a normal web page. That's it — no setup.

### Option B — Host it on GitHub Pages
1. Push `csv-dashboard-builder.html` to a GitHub repository.
2. In the repo, go to **Settings → Pages**.
3. Under "Build and deployment," set the source branch (e.g., `main`) and root folder.
4. GitHub will publish a URL like `https://yourusername.github.io/your-repo/csv-dashboard-builder.html`.
5. Share that link — anyone can use the tool without installing anything.

> Note: An internet connection is required the first time the page loads, since PapaParse and Chart.js are pulled from a CDN.

---

## Step-by-step: using the tool

**Step 1.** Open the HTML file in a browser.

**Step 2.** Click the yellow **Get Data** button in the top ribbon (or the **Get Data** button on the empty canvas).

**Step 3.** In the file picker, select one or more `.csv` files, then confirm.

**Step 4.** Wait a moment — the tool parses each file and lists its columns in the **Fields** pane on the left, grouped by file name.

**Step 5.** A first visual (a bar chart) is added automatically. Click any visual card on the canvas to select it — a yellow outline appears around it.

**Step 6.** With a visual selected, use the **Visualization** pane on the right to:
   - pick a chart type from the icon grid,
   - choose the data table (if you uploaded more than one CSV),
   - choose the **Axis** field (what you're grouping/plotting by),
   - choose one or more **Values** fields (what you're measuring),
   - choose how values are summarized (Sum, Average, Count, Min, Max),
   - pick a color from the swatch row.

**Step 7.** Click **+ New Visual** in the ribbon to add another chart, or **+ New Slicer** to add a dropdown filter.

**Step 8.** To remove a visual, hover its card and click the **✕** in its header.

**Step 9.** Toggle **Dark canvas** (top-right switch) to switch the report between light and dark themes.

---

## How the data pipeline works

1. **Upload** — When you select CSV files, each one is handed to PapaParse with `header: true`, which reads the first row as column names and returns an array of row objects.

2. **Type detection** — For each column, the tool samples up to 60 rows and checks:
   - If more than 80% of sampled values convert cleanly to numbers → the column is typed **number**.
   - Else if more than 70% look like valid dates → typed **date**.
   - Otherwise → typed **text**.

   This type feeds the icon shown in the Fields pane (Σ for numbers, a calendar glyph for dates, "Abc" for text) and determines which columns are offered as candidates for the **Values** field (numeric columns are prioritized).

3. **State** — All uploaded tables, their columns, and their rows are kept in a single in-memory JavaScript object (`state`). Nothing is written to disk or sent anywhere.

4. **Aggregation** — When a chart needs to plot values grouped by a category (e.g., total revenue by region), the tool groups rows by the Axis field's value, then applies your chosen aggregation (sum/average/count/min/max) to the Values field within each group.

5. **Rendering** — The aggregated labels and values are handed to Chart.js, which draws the chart onto a `<canvas>` element inside the visual card. Whenever you change a setting in the Visualization pane, the relevant chart is destroyed and redrawn with the new configuration.

---

## How visuals are configured

Each visual on the canvas is backed by a small settings object with fields like:

- `type` — the chart type (bar, line, area, pie, doughnut, scatter, kpi, table, or slicer)
- `table` — which uploaded CSV it reads from
- `axis` — the column used for categories/X-axis
- `values` — the column(s) used for measures/Y-axis
- `agg` — the aggregation method (sum, average, count, min, max)
- `colorIndex` — which color from the Power BI palette is applied

Selecting a visual on the canvas loads its settings into the right-hand Visualization pane, so changes you make there apply only to that visual.

---

## Chart types explained

| Type | Best for | Notes |
|---|---|---|
| **Bar chart** | Comparing totals across categories | Supports multiple Values fields (grouped bars) |
| **Line chart** | Trends over time or ordered categories | Supports multiple Values fields |
| **Area chart** | Trends with emphasis on volume/magnitude | Line chart with the area beneath it filled |
| **Pie chart** | Share of a whole across few categories | Single Values field only |
| **Donut chart** | Same as pie, with a hollow center | Single Values field only |
| **Scatter chart** | Relationship between two numeric fields | Axis and Values should both be numeric columns |
| **Card (KPI)** | A single summary number | No axis needed — shows one aggregated value |
| **Table** | Raw or lightly summarized rows | Shows up to 200 rows for performance |

---

## Slicers (filtering)

A **slicer** is a dropdown visual. When you add one:

1. Choose which data table it reads from.
2. Choose which column it filters on.
3. Selecting a value from its dropdown filters every other visual on the canvas that reads from the same table — mirroring how slicers behave in Power BI.

Set a slicer back to **(All)** to remove its filter.

---

## Color scheme

The tool uses Microsoft Power BI's brand and default report colors:

- **Brand accent:** `#F2C811` (Power BI yellow), used for the ribbon underline, primary buttons, and selection highlight.
- **Default data-color palette** (used across chart series, in order):
  `#01B8AA` `#374649` `#FD625E` `#F2C80F` `#5F6B6D` `#8AD4EB` `#FE9666` `#A66999` `#3599B8` `#DFBFBF`
- **Light canvas:** background `#F3F2F1`, cards `#FFFFFF`, text `#252423`.
- **Dark canvas:** background `#1E1E1E`, cards `#2B2B2B`, text `#F3F2F1`.
- **Font:** Segoe UI (with system-font fallbacks), matching Power BI Desktop's default typography.

---

## File structure

```
csv-dashboard-builder.html   ← the entire application (HTML + CSS + JS in one file)
README.md                    ← this file
```

There is no build process, package.json, or dependency install step — everything needed is either inline or pulled from a CDN at load time.

---

## Known limitations

- Designed for CSVs with a header row; files without headers will misread the first data row as column names.
- Type detection is sample-based (first ~60 rows) and may misclassify columns with mixed or unusual formatting.
- The Table visual caps display at 200 rows for performance; the underlying data is not limited, only the rendered preview.
- Large CSVs (very many rows) may slow down aggregation and chart redraws, since all processing happens in the browser.
- Requires an internet connection to load the PapaParse and Chart.js CDN scripts.

---

## Browser support

Works in current versions of Chrome, Edge, Firefox, and Safari. Requires JavaScript enabled. On narrow/mobile screens, the Fields and Visualization side panels are hidden to preserve canvas space.

---

## License / credit

**Built by Nova Agentic.**
Free to use, modify, and host. Third-party libraries (PapaParse, Chart.js) are used under their own respective open-source licenses.
