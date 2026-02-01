# SPEI Climate Data Downloader

Downloads historical and projected SPEI (Standardized Precipitation Evapotranspiration Index) climate data for US counties.

## Features

- Downloads historical SPEI data from West-Wide Drought Tracker (1895-present)
- Downloads GridMET SPEI data (1980-present)
- Downloads future SPEI projections from MACA climate models (2020-2099)
- Supports multiple counties with area-weighted averaging
- Outputs formatted Excel files with metadata

## Installation

1. Clone this repository
2. Create a virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Usage

Run the script:
```bash
./spei
```

Or directly:
```bash
source venv/bin/activate
python3 climate_spei_downloader.py
```

Follow the prompts to enter county name(s) and state.

## Output

The program creates Excel files in the `output/` directory with five tabs:
- **West-Wide Drought Tracker** - Raw historical data (1895-present)
- **GridMET** - Raw modern data (1980-present)
- **Climate Projections** - Raw projection data (2020-2099)
- **Drought indicators -- Historic** - Formatted historical data starting from 1970
- **Drought indicators -- Projected** - Formatted projections with 4 climate models

## Data Sources

- [West-Wide Drought Tracker](https://wrcc.dri.edu/wwdt/) - PRISM-based historical data
- [GridMET](http://www.climatologylab.org/gridmet.html) - High-resolution gridded data
- [MACA Climate Projections](https://climate.northwestknowledge.net/MACA/) - Downscaled future projections (RCP 4.5)

## Climate Models (Projections)

- CNRM-CM5
- CanESM2
- HadGEM2-ES365
- IPSL-CM5A-MR
