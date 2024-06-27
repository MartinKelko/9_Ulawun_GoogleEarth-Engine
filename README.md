# Ulawun Volcano Lava-Fountaining Visualization

## Description

This script utilizes Google Earth Engine to visualize the impacts of Ulawun volcano's lava-fountaining episode. By analyzing Sentinel-2 imagery from two different time periods, it provides a detailed view of the changes in the landscape due to volcanic activity. The visualization is created using a split panel interface to compare false color images from before and after the event.

## How to Use

### Prerequisites

- A Google Earth Engine account.
- Familiarity with the Google Earth Engine JavaScript API.

### Steps

1. **Define Coordinates and Date Ranges**

   The script defines the polygon coordinates and the date ranges for image acquisition.

   ```javascript
   // Define coordinates for the polygon
   var coords = [
     [151.206379, -5.09197],
     [151.206379, -4.997238],
     [151.359673, -4.997238],
     [151.359673, -5.09197],
     [151.206379, -5.09197]
   ];

   // Create a polygon geometry
   var polygon = ee.Geometry.Polygon(coords);

   // Define the date acquisitions
   var startDate_1 = '2023-09-01';
   var endDate_1 = '2023-09-03';
   var startDate_2 = '2023-12-20';
   var endDate_2 = '2023-12-22';

// Load the Sentinel-2 L1C/false color ImageCollection for the first date range
var sentinel1 = ee.ImageCollection('COPERNICUS/S2')
                 .filterDate(startDate_1, endDate_1)
                 .filterBounds(polygon)
                 .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 100)
                 .sort('system:time_start');

// Load the Sentinel-2 L1C/false color ImageCollection for the second date range
var sentinel2 = ee.ImageCollection('COPERNICUS/S2')
                 .filterDate(startDate_2, endDate_2)
                 .filterBounds(polygon)
                 .filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 100)
                 .sort('system:time_start');

// Define visualization parameters for false color images with enhanced saturation
var falseColorVis = {
  bands: ['B8', 'B4', 'B3'],  // False-color composite: NIR (8), Red (4), Green (3)
  min: 0,
  max: 4000,  // Increased max value for higher contrast
  gamma: 1.4  // Gamma correction to enhance saturation
};

// Create the maps
var map1 = ui.Map();
var map2 = ui.Map();

// Add the polygon as a layer to both maps
map1.addLayer(polygon, {color: 'red'}, 'Polygon');
map2.addLayer(polygon, {color: 'red'}, 'Polygon');

// Check if there is an image available for the first date range
if (sentinel1.size().getInfo() > 0) {
  var image1 = sentinel1.first();
  map1.centerObject(polygon, 13);
  map1.addLayer(image1, falseColorVis, 'Sentinel-2 False Color (Date Range 1)');
} else {
  map1.centerObject(polygon, 13);
  map1.addLayer(ee.Image().paint(polygon, 0, 2), {palette: 'red'}, 'No image available (Date Range 1)');
  print("No image available for the first date range and the specified criteria.");
}

// Check if there is an image available for the second date range
if (sentinel2.size().getInfo() > 0) {
  var image2 = sentinel2.first();
  map2.centerObject(polygon, 13);
  map2.addLayer(image2, falseColorVis, 'Sentinel-2 False Color (Date Range 2)');
} else {
  map2.centerObject(polygon, 13);
  map2.addLayer(ee.Image().paint(polygon, 0, 2), {palette: 'red'}, 'No image available (Date Range 2)');
  print("No image available for the second date range and the specified criteria.");
}

// Add the maps to the UI
var splitPanel = ui.SplitPanel({
  firstPanel: map1,
  secondPanel: map2,
  orientation: 'horizontal',
  wipe: true
});

// Display the split panel
ui.root.clear();
ui.root.add(splitPanel);

// Add map titles
map1.add(ui.Label('Sentinel-2 L1C False Color (Acquisition Date: 2 September 2023)'));
map2.add(ui.Label('Sentinel-2 L1C False Color (Acquisition Date: 21 December 2023)'));

