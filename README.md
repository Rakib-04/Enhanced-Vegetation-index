# Enhanced-Vegetation-index
The EVI actually allows for better differentiation of dense vegetation than the NDVI. This index minimizes saturation and background effects in the NDVI.
# At first define the Area of interest, it can be done through two ways. By uploading the shapefile or filter it from global database.
var dataset = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
    .filterDate('2023-01-01', '2023-06-01')
    .filterMetadata('CLOUD_COVER','less_than', 1);
    
// Applies scaling factors.
function applyScaleFactors(image) {
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  var thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
  return image.addBands(opticalBands, null, true)
              .addBands(thermalBands, null, true);
}

dataset = dataset.map(applyScaleFactors);

var visualization = {
  bands: ['SR_B4', 'SR_B3', 'SR_B2'],
  min: 0.0,
  max: 0.3,
};

// Map.setCenter(-114.2579, 38.9275, 8);
var boundary = dataset.median().clip(Roi);


Map.addLayer(boundary, visualization, 'True Color (432)');
Map.centerObject(Roi,10);

var ndvi_test= boundary.normalizedDifference(['SR_B5','SR_B4']).rename('NDVI');

var evi = boundary.expression(
  '2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))', {
    'NIR': boundary.select('SR_B5'),
    'RED': boundary.select('SR_B4'),
    'BLUE': boundary.select('SR_B2')
});

var color = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',
               '74A901', '66A000', '529400', '3E8601', '207401', '056201',
               '004C00', '023B01', '012E01', '011D01', '011301'];

Map.addLayer(ndvi_test, {min: 0, max: 1, palette: color},'NDVI_2023');

Map.addLayer(evi, {min: 0, max: 1, palette: color},'EVI_2023');

Export.image.toDrive({
  image:ndvi_test,
  description:'NDVI of Gazipur_2023',
  region:Roi,
  scale:30,
  crs:'EPSG:32646'
});

Export.image.toDrive({
  image:evi,
  description:'EVI of Gazipur_2023',
  region:Roi,
  scale:30,
  crs:'EPSG:32646'
});
