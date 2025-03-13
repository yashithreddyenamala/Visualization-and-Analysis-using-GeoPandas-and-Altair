# Visualization and Analysis using GeoPandas and Altair

This project involves data visualization and analysis using GeoPandas and Altair to explore taxi trip data in Chicago. The following tasks are implemented:


## Pre-Requisites
Before running the code, install the required libraries using the following command:
~~~bash
pip install geopandas altair pandas
~~~


## Task 1: Data Preparation
The dataset is read into a DataFrame, converted into a GeoDataFrame, and sampled for analysis.
~~~bash
import pandas as pd
import geopandas as gpd
import altair as alt
from shapely.geometry import Point
~~~
~~~bash
df = pd.read_csv('data/Taxi_Trips.csv')
geometry = [Point(xy) for xy in zip(df['Pickup Centroid Longitude'], df['Pickup Centroid Latitude'])]
gdf = gpd.GeoDataFrame(df, geometry=geometry, crs=4326).sample(1000)
gdf = gdf.rename(columns={"Trip Seconds": "Trip_Seconds", "Trip Miles": "Trip_Miles"})

gdf.head()
~~~


## Task 2: Data Joining and Cleaning
A geographic data file for Chicago is read and merged with the taxi trip data based on spatial location. The Fare column is converted to numeric, and null values are checked.
~~~bash
chicago = gpd.read_file('data/chicago.geojson')

joined = gpd.sjoin(gdf, chicago, predicate='within')
joined['Fare'] = pd.to_numeric(joined['Fare'], errors='coerce')
joined = joined[['zip', 'Fare']]
joined = joined.groupby('zip', as_index=False).mean()

print(joined.isna().sum())

merged = chicago.merge(joined, on='zip')
~~~


## Task 3: Interactive Heatmap Visualization Matrix
Heatmaps are created to visualize trip metrics (Trip_Seconds, Trip_Miles, and Fare) geographically using longitude and latitude.

### Heatmap Code 1: Heatmaps for Individual Metrics

The following heatmaps visualize individual metrics over geographic locations:
- Trip Seconds (Red Color Scale)
- Trip Miles (Blue Color Scale)
- Fare (Green Color Scale)

Each heatmap utilizes:
- Binned Longitude and Latitude for grouping data into grid cells.
- Color intensity to represent the selected metric values.
- Brushing to allow interactive filtering across multiple heatmaps.


~~~bash
import altair as alt

# Creating a selection interval(for brushing)
brush = alt.selection_interval()

# Heatmap for Trip_Seconds
heatmap_trip_seconds = alt.Chart(gdf).mark_rect().encode(
    x=alt.X('Pickup Centroid Longitude', bin=alt.Bin(maxbins=30)),  # Binned x-axis for longitude
    y=alt.Y('Pickup Centroid Latitude', bin=alt.Bin(maxbins=30)),   # Binned y-axis for latitude
    color=alt.Color('Trip_Seconds', scale=alt.Scale(scheme='reds')),    # Color intensity based on Trip_Seconds
    opacity=alt.condition(brush, alt.value(1), alt.value(0.1))  # Adjust opacity based on brush selection
).add_params(brush)

# Heatmap for Trip_Miles
heatmap_trip_miles = alt.Chart(gdf).mark_rect().encode(
    x=alt.X('Pickup Centroid Longitude', bin=alt.Bin(maxbins=30)),  # Binned x-axis for longitude
    y=alt.Y('Pickup Centroid Latitude', bin=alt.Bin(maxbins=30)),   # Binned y-axis for latitude
    color=alt.Color('Trip_Miles', scale=alt.Scale(scheme='blues')), # Color intensity based on Trip_Miles
    opacity=alt.condition(brush, alt.value(1), alt.value(0.1))  # Adjust opacity based on brush selection
).add_params(brush)

# Heatmap for Fare
heatmap_fare = alt.Chart(gdf).mark_rect().encode(
    x=alt.X('Pickup Centroid Longitude', bin=alt.Bin(maxbins=30)),  # Binned x-axis for longitude
    y=alt.Y('Pickup Centroid Latitude', bin=alt.Bin(maxbins=30)),   # Binned y-axis for latitude
    color=alt.Color('Fare', scale=alt.Scale(scheme='greens')),  # Color intensity based on Fare
    opacity=alt.condition(brush, alt.value(1), alt.value(0.1))  # Adjust opacity based on brush selection
).add_params(brush)

# Combining all heatmaps for viewing together
heatmap_trip_seconds & heatmap_trip_miles & heatmap_fare
~~~

<p>
  <img src="Heatmaps/Heatmaps for Individual Metrics.png" alt="Heatmap for Individual Metrics">
</p>




### Heatmap Code 2: Combined Heatmap with Repeated Variables

This visualization uses Altair's repeat feature to efficiently create a grid of heatmaps comparing each metric against the others.


~~~bash
# Create a selection interval (for brushing)
brush = alt.selection_interval()

# Creating the heatmap with repeated variables in both rows and columns
heatmap = alt.Chart(gdf).mark_rect().encode(
    alt.X(alt.repeat("column"), bin=alt.Bin(maxbins=30)),   # Binned x-axis for longitude
    alt.Y(alt.repeat("row"), bin=alt.Bin(maxbins=30)),  # Binned y-axis for latitude
    color=alt.Color(alt.repeat("column"), scale=alt.Scale(scheme='reds')),  # Color based on each variable
    opacity=alt.condition(brush, alt.value(1), alt.value(0.1))  # Adjust opacity based on brush selection
).add_params(brush).properties(
    width=150,
    height=150
).repeat(
    row=['Fare', 'Trip_Miles', 'Trip_Seconds'], # Variables for rows
    column=['Trip_Seconds', 'Trip_Miles', 'Fare']   # Variables for columns
)

heatmap
~~~

<p>
    <img src="Heatmaps/Combined Heatmap with Repeated Variables.png" alt="Combined Heatmap with Repeated Variables">
</p>


## Usage Instructions
1. Install the required dependencies.
2. Download the provided datasets: Taxi_Trips.csv and chicago.geojson.
3. Run each task code in a Jupyter Notebook or preferred Python IDE.
