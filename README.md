# Drought-Demand Model (DDM)

A tool for water utilities to model the relationship between drought severity and water demand. It downloads climate data, fits a statistical model to your utility's historical demand, and produces an interactive Excel workbook for projecting future demand under climate change scenarios.

## How It Works

The model predicts gallons per capita per day (GPCD) as a function of:

- **Population served** -- demographic driver
- **Drought severity** -- SPEI (Standardized Precipitation Evapotranspiration Index)
- **Time trend** -- secular changes in consumption patterns
- **Lagged drought effects** -- historical drought impacts with exponential decay over 10 years

The core equation is:

```
ln(GPCD) = a + b*(Year - BaseYear) + gamma*WeightedNegDI + epsilon*CurrentDrought
Demand (AFY) = GPCD * Population * 365 / 325,851
```

The output is an Excel workbook with live formulas -- you can change the population growth rate, select different climate models, or re-run Excel's Solver to re-fit parameters, and all results update automatically.

## Prerequisites

- Python 3.8+
- Microsoft Excel (to use the output workbooks and Solver)
- Internet connection (for downloading climate data)

## Installation

```bash
git clone <this-repo>
cd <this-repo>

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Quick Start

There are two steps: (1) download SPEI climate data for your county, then (2) generate the drought-demand model using that data plus your utility's historical demand.

### Step 1: Download SPEI Data

You must run the SPEI downloader first. This downloads historical and projected drought index data for the county (or counties) your utility serves.

```bash
./spei
```

The program will prompt you for:

- **County name** (e.g., "Marin", "Los Angeles")
- **State** (e.g., "California")
- Optionally, additional counties for area-weighted averaging

It downloads data from three sources:

| Source | Period | Description |
|--------|--------|-------------|
| [West-Wide Drought Tracker](https://wrcc.dri.edu/wwdt/) | 1895--present | PRISM-based historical SPEI |
| [GridMET](http://www.climatologylab.org/gridmet.html) | 1980--present | High-resolution gridded SPEI |
| [MACA Climate Projections](https://climate.northwestknowledge.net/MACA/) | 2020--2099 | Downscaled future projections (RCP 4.5) |

The climate projections include four models: CNRM-CM5, CanESM2, HadGEM2-ES365, and IPSL-CM5A-MR.

**Output:** An Excel file in `output/` (e.g., `output/Marin_CA_SPEI.xlsx`) with five tabs:

1. West-Wide Drought Tracker (raw)
2. GridMET (raw)
3. Climate Projections (raw)
4. Drought indicators -- Historic (formatted, 1970--present)
5. Drought indicators -- Projected (formatted, 2020--2099)

The first run downloads a ~457 MB NetCDF file that is cached locally for subsequent runs.

### Step 2: Prepare Your Demand Data

Create a CSV file with four columns:

```
Year, Population Served, Demand A (AF), Demand A+B (AF)
1960, 124375, 19399, 19399
1961, 128786, 20166, 20166
...
2024, 196605, 22371, 22371
```

| Column | Description |
|--------|-------------|
| Year | Calendar year |
| Population Served | Number of people served by the utility |
| Demand A | A subset of demand, e.g., potable water production (acre-feet) |
| Demand A+B | A superset of demand, e.g., potable + recycled (acre-feet) |

If you only have one demand category, put the same values in both demand columns. The column headers you use are preserved in the output workbook as dropdown options.

Place the file in the `Input files/` directory.

### Step 3: Generate the Model

```bash
./drought-model
```

The interactive script will prompt you for:

| Prompt | Default | Description |
|--------|---------|-------------|
| SPEI file | -- | Select from files in `output/` |
| Demand file | -- | Select from files in `Input files/` |
| Sheet prefix | LA | Short abbreviation for sheet names (e.g., MMWD, LA) |
| Utility name | LADWP | Full name used in labels |
| Output file | `{utility}_drought_model.xlsx` | Output filename |
| Population CAGR | 0.001212 | Compound annual growth rate for projections |
| Projection end year | 2050 | Last year of future projections |
| Test period start | 2015 | Year where out-of-sample testing begins |
| MAPE start year | auto | First year for error calculation (default: 10 years after data start) |
| Skip auto-fitting | No | Whether to use default parameters instead of fitting |

The model automatically fits parameters to your data using L-BFGS-B optimization, then generates the Excel workbook and opens it.

## Output Workbook

The generated Excel file contains 11 sheets:

| Sheet | Contents |
|-------|----------|
| Explanation | Data source documentation |
| Assumptions | Adjustable inputs: population CAGR, climate model selector, demand type dropdown |
| Results | Summary table comparing demand in 2000, present, and projection year; MAPE metrics |
| Data for charts | Reference formulas for creating charts |
| Drought indicators -- Historic | SPEI data (1970--present) |
| Drought indicators -- Projected | SPEI projections (2020--2099) from 4 climate models |
| {PREFIX} Demand & Pop | Your historical data plus population projections |
| {PREFIX} Params | Model parameters, bounds, and pre-configured Solver setup |
| {PREFIX} TS | In-sample time series with all intermediate calculations |
| {PREFIX} Test | Out-of-sample test period for model validation |
| {PREFIX} Future | Full projection (historical + future) using selected climate model |

All cells use live Excel formulas. Change any assumption and the entire workbook recalculates.

## Advanced Usage

### Command-Line Interface

You can also run the model generator directly with command-line arguments:

```bash
source venv/bin/activate

python3 "Drought-demand model/generate_drought_demand_model.py" \
    --spei-file output/Marin_CA_SPEI.xlsx \
    --demand-file "Input files/MMWD pop + demand.csv" \
    --output "Drought-demand model/MMWD_drought_model.xlsx" \
    --prefix MMWD \
    --utility-name "Marin Municipal Water District" \
    --cagr 0.001212 \
    --projection-end 2050 \
    --test-start 2015
```

Add `--no-fit` to skip automatic parameter fitting and use the default (or specified) parameters.

### Re-fitting Parameters in Excel

The output workbook includes a pre-configured Excel Solver setup on the Params sheet. To re-fit:

1. Open the workbook in Excel
2. Go to the Params sheet
3. Run Solver (Data > Solver) -- the objective, variable cells, and constraints are pre-loaded
4. Solver will minimize MAPE by adjusting the five model parameters

## Project Structure

```
.
├── spei                          # Step 1: SPEI data download script
├── climate_spei_downloader.py    # SPEI downloader source code
├── drought-model                 # Step 2: Model generation script
├── Drought-demand model/
│   └── generate_drought_demand_model.py  # Model generator source code
├── Input files/                  # Place your demand/population CSV here
├── output/                       # SPEI downloads go here (generated)
├── requirements.txt              # Python dependencies
└── LICENSE                       # GPL v3
```

## Data Sources

- [West-Wide Drought Tracker](https://wrcc.dri.edu/wwdt/) -- PRISM-based historical drought data
- [GridMET](http://www.climatologylab.org/gridmet.html) -- High-resolution gridded meteorological data
- [MACA Climate Projections](https://climate.northwestknowledge.net/MACA/) -- Downscaled CMIP5 climate model projections

## License

This project is licensed under the GNU General Public License v3.0. See [LICENSE](LICENSE) for details.
