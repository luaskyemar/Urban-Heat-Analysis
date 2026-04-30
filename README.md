# Project Title

## Overview
Briefly describe the research question or goal.

### Repository Structure

- `data/`: input data
- `notebooks/analysis.ipynb`: main analysis notebook
- `outputs/`: generated figures/tables

© Setup

Clone the repository:

```bash
git clone <https://github.com/luaskyemar/Urban-Heat-Analysis>
cd my-project

## Data

The Landsat raster is stored in:

data/raw/Landsat/LandsatComposite_Zurich_1985.tif

## Data loading

The raster is loaded using a relative file path to ensure reproducibility.
Key properties:
- Multiple spectral bands
- CRS: [insert CRS]
- Spatial extent: [brief description]

## Indices

### NDVI
NDVI is calculated using the red and near-infrared bands. (3 red, 4 NIR)
Calculated using:
- Band 3 (Red)
- Band 4 (NIR)

### LST
Land Surface Temperature is derived from the thermal infrared band (Band 6 in Landsat 5), following standard radiometric conversion steps.
Derived from:
- Band 6 (Thermal infrared)