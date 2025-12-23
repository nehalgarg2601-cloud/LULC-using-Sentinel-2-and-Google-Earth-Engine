# LULC-using-Sentinel-2-and-Google-Earth-Engine
This project demonstrates supervised land use/land cover classification of Sentinel-2 imagery using a Random Forest classifier in Google Earth Engine

// -----------------------------------------------------
// Load and filter Sentinel-2 data
// -----------------------------------------------------

var image = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(geometry)
  .filterDate('2023-01-01', '2023-12-31')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
  .select(['B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8'])
  .median();

// -----------------------------------------------------
// Add NDVI (Normalized Difference Vegetation Index)
// -----------------------------------------------------

var ndvi = image.normalizedDifference(['B8', 'B4']).rename('NDVI');
image = image.addBands(ndvi);

// -----------------------------------------------------
// Assign class labels to training samples
// -----------------------------------------------------

sand = sand.map(function (f) {
  return f.set('class', 0);
});

vegetation = vegetation.map(function (f) {
  return f.set('class', 1);
});

urban = urban.map(function (f) {
  return f.set('class', 2);
});

// Merge all training samples
var sample = sand.merge(vegetation).merge(urban);

// Bands used for classification
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'NDVI'];

// -----------------------------------------------------
// Sample image at training locations
// -----------------------------------------------------

var training = image.select(bands).sampleRegions({
  collection: sample,
  properties: ['class'],
  scale: 10
});

// -----------------------------------------------------
// Train Random Forest classifier
// -----------------------------------------------------

var classifier = ee.Classifier.smileRandomForest(100).train({
  features: training,
  classProperty: 'class',
  inputProperties: bands
});

// -----------------------------------------------------
// Apply classifier and clip to Area of Interest
// -----------------------------------------------------

var classified = image.select(bands).classify(classifier);
var classified_clipped = classified.clip(geometry);

// -----------------------------------------------------
// Display results
// -----------------------------------------------------

Map.centerObject(geometry, 13);

Map.addLayer(classified_clipped, {
  min: 0,
  max: 2,
  palette: ['yellow', 'green', 'blue']
}, 'LULC Classification');

