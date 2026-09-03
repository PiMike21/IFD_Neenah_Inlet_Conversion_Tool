# Neenah Inlet Flow Curve Explorer

A standalone browser-based reference and calculation tool for selecting **Neenah inlet configurations** and generating **Autodesk InfoDrainage rated-by-flow curves**. The tool combines an embedded inlet catalog with interactive flow-curve visualization, hydraulic recalculation, QA metadata, and CSV export.

---

## Overview

Neenah Inlet Flow Curve Explorer provides a searchable catalog of 155 inlet configurations. Each configuration includes a Neenah catalog identifier, description, component code, K coefficient, source location, QA status, and a rated-by-flow dataset.

The default curves use an on-grade, uniform triangular gutter-flow basis with a 2% longitudinal slope, 2% road/gutter cross slope, Manning's n of 0.016, and gutter depths from 0 to 0.30 ft. Users can inspect captured flow, bypass flow, and capture efficiency, then export the active curve as a two-column CSV for InfoDrainage.

Everything is contained in a single static HTML file, including the catalog data, CSS, JavaScript, hydraulic calculations, SVG chart rendering, and file export logic.

---

## Features

### Neenah Inlet Catalog

- 155 embedded inlet configurations
- Search by catalog number, description, component code, or internal configuration ID
- Configuration selector showing catalog, description, and component code
- Source PDF page and panel location for each inlet
- K coefficient display for the active configuration
- Maximum captured flow summary
- QA confidence badge and review notes where applicable
- Direct link to the Neenah grate-capacity source PDF

### Flow Curve Visualization

The interactive SVG chart plots:

- **Approach flow** on the horizontal axis
- **Captured flow** as the primary curve
- **Bypass flow** as an optional secondary curve

Mouse and touch interaction displays the nearest data point with:

- Approach flow
- Captured flow
- Bypass flow
- Capture efficiency

The chart automatically redraws when the browser is resized or when hydraulic inputs change.

### Rated-by-Flow Table

Each active curve is displayed as a table containing:

| Column | Description |
| --- | --- |
| Point | Sequential curve point number |
| Approach flow | Total gutter approach flow in cfs |
| Captured flow | Flow captured by the inlet in cfs |
| Bypass flow | Approach flow not captured by the inlet in cfs |
| Capture efficiency | Captured flow divided by approach flow, expressed as a percentage |

### InfoDrainage CSV Export

The **Download InfoDrainage CSV** command creates a two-column file containing:

```csv
Approach Flow,Captured Flow
0.000000,0.000000
...
```

Only **Approach Flow** and **Captured Flow** are included in the exported file, matching the rated-by-flow import structure used by the tool for InfoDrainage.

Catalog-basis calculations use the embedded CSV filename assigned to the selected inlet. Custom calculations append the active longitudinal slope, cross slope, Manning's n, and K coefficient to the filename.

### Copy Rating Table

The **Copy rating table** command copies the active approach-flow and captured-flow columns as tab-separated text. It uses the browser Clipboard API when available and includes a fallback copy method for browsers where direct clipboard access fails.

---

## Hydraulic Basis

The catalog conversion uses a uniform triangular gutter-flow calculation.

### Approach Flow

```text
Q = (0.56 / n) × (1 / ST) × D^(8/3) × √SL
```

Where:

- `Q` = approach flow
- `n` = Manning's roughness coefficient
- `ST` = gutter transverse slope
- `D` = gutter depth
- `SL` = longitudinal slope

### Captured Flow

```text
Qc = min(Q, K × D^(5/3))
```

Where:

- `Qc` = captured flow
- `K` = Neenah inlet coefficient

### Bypass Flow

```text
Qb = max(0, Q - Qc)
```

### Capture Efficiency

```text
Efficiency = Qc / Q × 100
```

Zero approach flow is reported as 100% efficiency.

---

## Catalog Conversion Basis

| Parameter | Default |
| --- | ---: |
| Longitudinal slope | 2.0% |
| Road cross slope | 2.0% |
| Gutter cross slope | 2.0% |
| Manning's n | 0.016 |
| Maximum gutter depth | 0.30 ft |
| Depth increment | 0.01 ft |
| Flow units | cfs |

Each embedded catalog curve contains 31 rated-flow points over the default 0 to 0.30 ft depth range.

The catalog curves are limited to **on-grade, uniform triangular gutter flow**. They are not ponded or sag-inlet curves.

---

## Custom Hydraulic Calculations

The hydraulic input panel can regenerate the selected inlet curve using modified hydraulic parameters.

Editable inputs include:

- Longitudinal slope
- Road cross slope
- Gutter cross slope
- Manning's n
- Maximum depth
- Depth increment
- K coefficient source
- Neenah K coefficient

### K Coefficient Modes

**Catalog basis K** uses the K value embedded with the selected inlet and is intended for the verified 2% longitudinal / 2% transverse catalog basis.

**User-supplied, slope-verified K** enables manual K entry. A user-supplied K is required whenever the longitudinal or gutter cross slope differs from the 2% catalog basis.

Changing an input marks the calculation as pending until **Apply inputs** is selected. **Reset basis** restores the selected inlet to its embedded catalog basis.

---

## Input Validation and Limits

Custom calculations enforce the following limits:

| Input | Accepted Range |
| --- | --- |
| Longitudinal slope | Greater than 0% and no more than 20% |
| Road cross slope | Greater than 0% and no more than 20% |
| Gutter cross slope | Greater than 0% and no more than 20% |
| Manning's n | 0.005 to 0.100 |
| Maximum depth | Greater than 0 ft and no more than 2 ft |
| Depth increment | 0.001 ft to 0.10 ft |
| K coefficient | Greater than 0 |
| Maximum generated points | 501 |

Additional hydraulic constraints:

- The depth increment cannot exceed the maximum depth.
- Road and gutter cross slopes must match.
- Composite or depressed gutter geometry is not supported by the custom calculator.
- Custom slopes require a user-supplied Neenah K coefficient verified for the selected inlet and slope configuration.

Custom calculations are identified separately from the verified catalog basis in the interface and export filename.

---

## Catalog Data and QA Metadata

The embedded JSON data identifies its conversion source as:

```text
Neenah_InfoDrainage_Conversion.xlsx
```

The catalog contains configuration metadata including:

- Catalog identifier
- Description
- Component code
- Chart page
- Source PDF page
- Source panel location
- K coefficient and raw K value
- Confidence status
- Header status
- Plot-detection status
- Source panel image name
- CSV filename
- Review notes
- Rated-flow points
- Maximum approach, captured, and bypass flow

QA metadata across the 155 embedded configurations includes:

| QA Field | Count |
| --- | ---: |
| High confidence | 151 |
| Manual visual review | 4 |
| Plot detected | 145 |
| Plot detection fallback | 10 |
| Header status: Good | 155 |

Configurations requiring review are visually distinguished in the interface, and review notes are exposed through the QA badge.

---

## Source Reference

The **Review source** control opens the Neenah grate-capacity source PDF:

[Neenah Inlet / Grate Capacities PDF](https://nasecawi.org/wp-content/uploads/2021/02/Neenah-Inlet_Grate_Capacities-2.pdf)

Each selected configuration displays its corresponding PDF page and panel location. The embedded catalog spans source PDF pages 5 through 24 and chart pages 1 through 39.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Application | Standalone HTML |
| Styling | Embedded CSS |
| Logic | Vanilla JavaScript |
| Catalog data | Embedded JSON |
| Charting | Browser SVG generated with JavaScript |
| CSV export | Blob + Object URL browser APIs |
| Clipboard | Clipboard API with fallback copy logic |
| Backend | None |
| External JavaScript libraries | None |

No build system, package manager, framework, or server is required by the HTML file.

---

## Running the Tool

No installation is required.

1. Save `Neenah_Inlet_Webapp.html` locally.
2. Open the file in a modern web browser.
3. Search for or select a Neenah inlet configuration.
4. Review the catalog curve, source metadata, chart, and rated-by-flow table.
5. Optionally adjust the hydraulic inputs and apply the recalculation.
6. Download the active curve as an InfoDrainage CSV or copy the rating table to the clipboard.

---

## Privacy and Local Operation

The tool runs locally in the browser and does not use a backend. Catalog data and hydraulic calculations remain local, and CSV files are generated with browser APIs.

The external **Review source** link opens the Neenah PDF when selected. Catalog browsing, calculations, charting, and CSV generation perform no data uploads.

---

## Responsive Interface

The layout adapts for smaller browser widths:

- The two-column workspace collapses to a single column below 1120 px.
- Summary cards collapse from four columns to two below 860 px.
- Hydraulic inputs, statistics, and action controls collapse to single-column layouts below 700 px.
- The chart height is reduced on narrow screens.
- Reduced-motion preferences disable button hover motion.

---

## File Structure

The complete tool is contained in one file:

```text
Neenah_Inlet_Webapp.html
├── Embedded CSS
├── Application interface
├── Embedded Neenah catalog JSON
├── Hydraulic calculation logic
├── SVG chart rendering
├── Search and selection logic
├── QA/source metadata display
├── CSV generation
└── Clipboard export
```

