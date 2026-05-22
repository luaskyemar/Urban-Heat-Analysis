# Spatiotemporal Analysis of LST and NDVI in Zurich (1985–2024)
This workflow processes tri-annual multispectral data from 1985 to 2024. For each year, it computes the Normalized Difference Vegetation Index (NDVI) and Land Surface Temperature (LST), and organizes the results into a multi-year `xarray` datacube. All subsequent analyses are based on this datacube and aim to answer the following research question:

How have Land Surface Temperatures (LST) changed in Zurich between 1985 and 2024, and how do these changes relate to changes in NDVI?

## Data 
The data are stored as GeoTIFF files in data/raw/Landsat/ and consist of Landsat composites for 14 years between 1985 and 2024. 
Each file contains surface reflectance and thermal bands needed to compute NDVI and LST.

## Workflow Overview

1. Data Preparation  
   - Load Landsat scenes  
   - Compute NDVI and LST for each year
   - Combine all yearly scenes into a multi-year data cube  

2. Trend Analysis  
   - Compute mean NDVI and LST trends over time  
   - Visualize spatial patterns for start and end years  

3. Change Detection  
   - Compute pixel-wise change between first and last year  
   - Generate change maps  
   - Estimate linear trends per decade  

4. Correlation Analysis  
   - Compute pixel-wise NDVI–LST correlation  
   - Apply significance masking  
   - Analyze correlation evolution over time  
   - Compute overall correlation across all years  

## Project Structure

```bash
Urban-Heat-Analysis/
│
├── data/
│   └── raw/Landsat/         # Input Landsat composites
├── notebooks/
│   └── urban_heat_analysis.ipynb  # Main analysis notebook
|   └── RGB.ipynb            # RGB composites for the report 
├── outputs/                 # Exported plots                   # 
└── README.md                # Project documentation

```
Clone the repository:

```bash
git clone <https://github.com/luaskyemar/Urban-Heat-Analysis>
cd Urban-Heat-Analysis
```

## Usage
Execute the Jupyter Notebook `urban_heat_analysis.ipynb` from top to bottom. Trend maps will be exported to `outputs`.

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
### Data Preparation  
#### Load the Scenes 
A list of file paths pointing to each Landsat composite you want to process is created. Each file corresponds to one year in your time series. Then a list of the actual years that match the files is created. 
Then the list of integers is converted into proper datetime objects, which xarray understands as a time dimension.

#### Compute LST and NDVI using a function
Each Landsat scene is processed using the function `process_scene()`.
The function:
- opens the raster file with `xarray`
- extracts the red, near-infrared, and thermal bands (the band selection is based on the following structure: Band 3 = Red, Band 4 = Near- infrared (NIR), Band 7 = Thermal)
- calculates NDVI
- estimates land surface emissivity from NDVI
- calculates Land Surface Temperature in Celsius
- returns an `xarray.Dataset` containing NDVI and LST

#### Build the Datacube 
Each scene is processed individually, assigned a time coordinate, and then combined into one data cube.
The resulting object, `ds_cube`, contains NDVI and LST values, stored as data variables, for every available year.
The data cube contains:
- dimensions: time, x,y, 
- coordinates: time, year (same thing, but time is in the appropraite format), y, x, spatial_ref, band
- variables: ndvi, lst

### Trend Analysis  
#### Compute Mean NDVI and LST Over Time
This calculates the spatial mean of LST and NDVI for each year in the datacube.
By averaging across the x and y dimensions, we obtain a single representative value per year, which is used to analyze long‑term temporal trends.

#### Plot Temporal Trends of LST and NDVI
This figure shows how mean NDVI and mean LST evolved between 1985 and 2024. LST is plotted on the left y‑axis (red), while NDVI is plotted on the right y‑axis (green). Both share the same x‑axis (years). This dual‑axis plot provides an overview of NDVI and LST dynamics over time.

#### Histogram Analysis for 1985 and 2024
The following block generates four histograms: LST distribution 1985, LST distribution 2024, NDVI distribution 1985, NDVI distribution 2024.

To understand how NDVI and LST values are distributed in the earliest and latest years, histograms were generated for 1985 and 2024. These distributions help reveal the overall range of values, highlight potential outliers, and guide the selection of appropriate clipping thresholds for map visualization (e.g., LST between 20–40 °C and NDVI between 0–0.8). Establishing these thresholds ensures that the final spatial maps remain visually interpretable and are not distorted by extreme or rare pixel values.

#### Spatial Maps of NDVI and LST (1985 vs 2024)
The following block generates four spatial maps: LST 1985, LST 2024, NDVI 1985, NDVI 2024.

Each map uses manually selected visualization ranges: 
LST: vmin=20, vmax=40
NDVI: vmin=0, vmax=0.8
These ranges were chosen based on the histogram analysis above.
The maps allow visual comparison of: LST changes, NDVI changes across Zurich. 
This step provides the spatial context needed before performing change detection and correlation analysis and already allows to detect correlations by eye. 

### Change Detection
#### Compute Change Between 1985 and 2024
These steps extract the LST and NDVI layers for the first and last years in the dataset and compute pixel‑wise differences.
The resulting rasters (lst_change and ndvi_change) show how temperature and vegetation have changed spatially over nearly four decades.
#### Histogram Analysis of Change Values
Histograms of NDVI and LST change values are generated to understand the distribution of changes and identify whether changes are mostly positive or negative. It also guides the selection of visualization ranges for the maps. This step provides a statistical overview of how much Zurich has warmed and how vegetation has shifted between 1985 and 2024.
#### Spatial Maps of NDVI and LST Change
Two spatial maps are produced: 

Δ LST (1985–2024) using a diverging red‑blue colormap, where blue indicates cooling and red indicates warming. The range was clipped to −5 to +5 °C for interpretability. 

Δ NDVI (1985–2024) using a vegetation‑friendly (RdYlGn) colormap, where red indictates a decrease in NDVI and green indicates an increase in NDVI, yellow being neutral. The range was clipped to −0.2 to +0.2

These maps reveal where the most significant warming and vegetation changes occurred, allowing spatial patterns to be compared directly.

#### Pixel-wise Linear Trends
A linear regression is fitted independently for every pixel across the time dimension. The slope of this regression represents the rate of change in LST and NDVI over time. Because xarray stores time in nanoseconds, the slope is converted to: degrees per year (1e9 * 60 * 60 * 24 * 365.25), and then degrees per decade (*10), which is a better standard to detect long term trends.
#### Histogram Analysis of Trends per Decade
Histograms of NDVI and LST trends per decade are generated to understand the distribution of trands and identify whether changes are mostly positive or negative. It also guides the selection of visualization ranges for the maps. This step provides a statistical overview of how much Zurich has warmed and how vegetation has shifted per decade between 1985 and 2024.

#### Visualizing NDVI and LST Trends per Decade 
LST Trend per Decade uses a diverging red–blue colormap, where blue indicates cooling and red indicates warming. The range was clipped to 0 to 2 °C due to the distribution. It shows long‑term temperature change patterns across Zurich. 

NDVI Trend per Decade uses a green–yellow–red vegetation colormap, where red indictates a decrease in NDVI and green indicates an increase in NDVI, yellow being neutral. The range was clipped to −0.1 to +0.1. It shows long‑term NDVI change patterns across Zurich. 

The robust=True argument ensures that extreme outliers do not distort the color scale.

### Correlation Analysis
#### Pixel‑wise Correlation Between NDVI and LST
A pixel‑wise Pearson correlation is computed between NDVI and LST across the full 1985–2024 time series.
For each pixel location, this calculation measures how vegetation and surface temperature co‑vary over time:

Negative correlation → higher NDVI associated with lower LST (vegetation cooling effect)
Positive correlation → higher NDVI associated with higher LST
Near zero → weak or no relationship

This produces a spatial correlation map (corr_map) that reveals where vegetation most strongly influences surface temperature.

#### Visualizing the Correlation Map
The correlation map is displayed using a diverging Purple-Orange colormap:

Orange = strong negative NDVI–LST relationship

Purple = strong positive relationship

White = weak or no correlation

The color scale is fixed between −1 and +1 to represent the full possible correlation range.

### Significance Testing
The first step converts each correlation value into a t‑statistic using the standard formula for Pearson correlation significance.
Then a two‑tailed p‑value is computed for each pixel.
Only correlations with p < 0.05 are retained. All non‑significant pixels are masked out.
This step ensures that the final map highlights only statistically robust NDVI–LST relationships.
The resulting map shows: where NDVI and LST are significantly linked, where vegetation has a consistent cooling effect, and where the relationship is weak or statistically unreliable. This provides a rigorous spatial assessment of the NDVI–LST interaction across Zurich.

### Correlation over Time 
To understand how the NDVI–LST relationship evolves over time, the correlation is computed separately for each year in the datacube.
Instead of manually selecting a few years, the workflow automatically loops through all available time steps:
First it extracts each year from the datacube, then it flattens the LST and NDVI values. This is done by taking a 2‑D raster (rows × columns) and turning it into a 1‑D vector (a long list of pixel values). Then the loop computes the Pearson correlation coefficient (r) and the proportion of LST variance explained by NDVI (R²), and stores the results in a table for comparison. 
This produces a year‑by‑year correlation timeline, allowing to track how the vegetation–temperature relationship has changed over four decades.
### Overall NDVI–LST Relationship (All Years Combined)
First the LST and NDVI values are flattened. From this it takes a random subset of 20,000 points to keep the scatterplot readable. Then a linear regression model is fitted to quantify the overall NDVI–LST trend. Then r and R² are computed to summarize the global strength of the relationship. As a final step,  a scatterplot with a regression line is created to visualize how NDVI and LST co‑vary across Zurich over the full 1985–2024 period.