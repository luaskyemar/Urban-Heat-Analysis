# Project Title
Zurich Urban Heat Analysis

## Overview and Research Question
This workflow processes tri-annual multispectral data from 1985 to 2024. It calculates NDVI and Land Surface Temperature (LST) for each year and combines the results into a single time-series data cube using `xarray`.

## Data 
Landsat composites for 14 years between 1985 and 2024
Stored as GeoTIFF files in data/raw/Landsat/
Each file contains surface reflectance and thermal bands needed to compute NDVI and LST

## Workflow Overview

The workflow consists of three main steps:

1. NONIG FIX
2. Process each Landsat scene to calculate NDVI and LST  
3. Combine all yearly scenes into a single data cube
## Repository Structure

Urban-Heat-Analysis/
│
├── data/
│   └── raw/Landsat/         # Input Landsat composites
├── notebooks/
│   └── analysis.ipynb       # Main analysis notebook
├── outputs/  # Exported plots
├── src/ 
└── README.md


Clone the repository:

```bash
git clone <https://github.com/luaskyemar/Urban-Heat-Analysis>
cd my-project
```
## Required Python Packages 

```python
import pandas as pd
import rasterio
import matplotlib.pyplot as plt
import numpy as np
import matplotlib.ticker as ticker
import scipy.stats as stats
```

## Workflow Summary 
### Prepare the data 
#### Load the Scenes 
files = ["../data/raw/Landsat/LandsatComposite_Zurich_1985.tif", ...]
This creates a list of file paths pointing to each Landsat composite you want to process.
Each file corresponds to one year in your time series. 

years = [1985, 1988, ..., 2024]
This is the list of actual years that match the files above.

times = pd.to_datetime(years)
This converts the list of integers into proper datetime objects, which xarray understands as a time dimension.

#### Compute LST and NDVI using a function
Each Landsat scene is processed using the function `process_scene()`.
The function:

- opens the raster file with `xarray`
- extracts the red, near-infrared, and thermal bands (The band selection assumes the following structure: Band 3 = Red, Band 4 = Near- infrared (NIR), Band 7 = Thermal)
- calculates NDVI
- estimates land surface emissivity from NDVI
- calculates Land Surface Temperature in Celsius
- returns an `xarray.Dataset` containing NDVI and LST

#### Build the Datacube 
Each scene is processed individually, assigned a time coordinate, and then combined into one data cube.
The resulting object, `ds_cube`, contains NDVI and LST values, stored as data variables, for every available year.
The data cube contains:
- variables: ndvi, lst
- coordinates: time, year (same thing, but year is in the appropraite format. 

### First Visualizations of Change over Time
#### Compute Mean NDVI and LST Over Time
This calculates the spatial mean of LST and NDVI for each year in the datacube.
By averaging across the x and y dimensions, we obtain a single representative value per year, which is used to analyze long‑term temporal trends.

#### Plot Temporal Trends of LST and NDVI
This figure shows how mean NDVI and mean LST evolved between 1985 and 2024.

LST is plotted on the left y‑axis (red)

NDVI is plotted on the right y‑axis (green)

Both share the same x‑axis (years)

This dual‑axis plot provides a high‑level overview of vegetation and temperature dynamics over time.

#### Histogram Analysis for 1985 and 2024
The following block generates four histograms:

LST distribution in 1985

LST distribution in 2024

NDVI distribution in 1985

NDVI distribution in 2024

To understand how NDVI and LST values are distributed in the earliest and latest years, histograms were generated for 1985 and 2024. These distributions help reveal the overall range of values, highlight potential outliers, and guide the selection of appropriate clipping thresholds for map visualization (e.g., LST between 20–40 °C and NDVI between 0–0.8). Establishing these thresholds ensures that the final spatial maps remain visually interpretable and are not distorted by extreme or rare pixel values.

#### Spatial Maps of NDVI and LST (1985 vs 2024)
The following block generates four spatial maps:

LST 1985

LST 2024

NDVI 1985

NDVI 2024

Each map uses manually selected visualization ranges: 
LST: vmin=20, vmax=40
NDVI: vmin=0, vmax=0.8

These ranges were chosen based on the histogram analysis above.
The maps allow visual comparison of: warming patterns, vegetation changes, spatial heterogeneity across Zurich. 
This step provides the spatial context needed before performing change detection and correlation analysis.