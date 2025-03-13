# Linked-Visualization


This project demonstrates how to create interactive heatmaps using the Altair visualization library. The heatmaps visualize trip data with multiple metrics such as Trip_Seconds, Trip_Miles, and Fare, mapped geographically using longitude and latitude.


# Pre-Requisites
Before running the code, ensure you have the following libraries installed:
~~~bash
pip install altair pandas geopandas
~~~

# Data Preparation
The provided code assumes a GeoDataFrame (gdf) containing the following columns:
- Pickup Centroid Longitude
- Pickup Centroid Latitude
- Trip_Seconds
- Trip_Miles
- Fare


# Code Overview

**Heatmaps for Individual Metrics**

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




**Combined Heatmap with Repeated Variables**

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


# Usage Instructions
1. Load your data into a GeoDataFrame.
2. Copy the provided code into your Python environment.
3. Run the code to visualize interactive heatmaps with brushing capabilities.
