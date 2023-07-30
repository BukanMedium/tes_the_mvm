---
layout: post
read_time: true
show_date: true
title:  Unsupervised Classification in Google Earth Engine
date:   2023-07-30 14:54:20 
description: To share my little knowledge abot GEE
img: posts/20230730/GEE-falsecolor.png
tags: [coding, javascript, GEE, Sentinel-2]
author: Shlhn
github: 
---

```js

// BISMILLAHIRRAHMANNIRRAHIM
// Content:
// 1. Image Preparation
// 2. Image Processing
// 3. Image Visualization
// 4. Image Classification
// 5. Exporting Data


// We will use the Sentinel-2 Surface Reflection product.
// When in doubt, see the GEE documentation. All of the function and dataset guide is available there


////////////////////////////////////////////////
//////       1. IMAGE PREPARATION        //////
//////////////////////////////////////////////

//first, draw a polygon geometry for the study area

//Display the geometry
Map.addLayer(geometry, {}, 'Area of Interest')

// This dataset has already been atmospherically corrected (surface reflectnce (SR))
var s2 = ee.ImageCollection("COPERNICUS/S2_SR");

// Function to mask clouds S2
function maskS2srClouds(image) {
  var qa = image.select('QA60');

  // Bits 10 and 11 are clouds and cirrus, respectively.
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;

  // Both flags should be set to zero, indicating clear conditions.
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));

  return image.updateMask(mask).divide(10000);
}

// Filter Sentinel-2 collection
//use filterBounds to filter for location
//use filterDate to filter for time
//use filterMetadata to filter for other variable provided in metadata
var s2Filt = s2.filterBounds(geometry)
                .filterDate('2016-01-01', '2023-07-13')
                .filterMetadata('CLOUDY_PIXEL_PERCENTAGE',
                                'less_than',15) 
                .map(maskS2srClouds);

print('Sentinel-2 Filtered collection',s2Filt);

// Composite images and clip
var s2composite = s2Filt.median().clip(geometry); 
//The median can be changed to mean, min, etc
//The clip feature (table) can be changed to geometry, etc



////////////////////////////////////////////////
//////       2. IMAGE PROCESSING         //////
//////////////////////////////////////////////

//Calculating NDVI, NNDWI, NDBI, NDBSI, and NDCI
//ND: Normalized Difference (to calculate normalized difference, use iamge.normalizedDifference(['Band1', 'Band2'])-
//-this will automatically use the formula (band1-band2)/(band1+band2)

//VI: Vegetation Index 
//CI: Chloropyhll Index 
//WI: Water Index
//BSI: Bare Soil Index
//BI: Built-up Index

//You can check it yourself here: https://custom-scripts.sentinel-hub.com/custom-scripts/sentinel/sentinel-2/

var NDVI = s2composite.normalizedDifference(['B8', 'B4']).rename('NDVI');
var NDWI = s2composite.normalizedDifference(['B3', 'B8']).rename('NDWI');
var NDBI = s2composite.normalizedDifference(['B11', 'B8']).rename('NDBI');
var NDBSI = s2composite.normalizedDifference(['B11', 'B5']).rename('NDBSI');
var NDCI = s2composite.normalizedDifference(['B5', 'B4']).rename('NDCI');


///////////////////////////////////////////////
///////// 3. IMAGE VISUALIZATION /////////////
/////////////////////////////////////////////

// Add composite to map
// Display true color (4,3,2)
Map.addLayer(s2composite,{bands:['B4','B3','B2'],min:0,max:0.3,
                          gamma:1.5},'True Color');

// Display false color (8,4,3)
Map.addLayer(s2composite,{bands:['B8','B4','B3'],min:0,max:0.3,
                          gamma:1.5},'False Color');


//Creating color palette for visualization
//To learn how the colors are coded, you can search for 'RRGGBB' color code
var palette = 'FF0000, FFFF00, 00FF00';

// Display NDVI
Map.addLayer(NDVI, {min:-1, max:1, palette: palette}, 'NDVI');

// Center to AOI
Map.centerObject(geometry, 9); //set the zoom by changing the number, higher number for more zoom in




///////////////////////////////////////////////
/////////// 4. IMAGE CLASSIFICATION //////////
/////////////////////////////////////////////



// Load a pre-computed composite image for input.
// create new variable of combined bands
var combined = ee.Image([NDVI, NDWI, NDBI, NDBSI, NDCI]);

// Define a region in which to generate a sample of the input.
var train_region = geometry2; //draw another geometry to train the classifier or use import assets

// Display the sample region.
Map.addLayer(ee.Image().paint(train_region, 0, 2), {shown: false}, 'train region');

// Make the training dataset.
var training = combined.sample({
  region: train_region,
  scale: 30,
  numPixels: 5000,
  seed: 111, //use any number
  });


// There are many methods to classify a dataset, pick one that best suits your needs
// Don't forget to check the docs
// Instantiate the clusterer and train it.
var clusterer = ee.Clusterer.wekaKMeans({
  nClusters: 15, 
  maxIterations: 100,
  seed: 111
  })
  .train(training);

// Cluster the input using the trained clusterer.
var result = combined.cluster(clusterer);

// Display the clusters with random colors.
Map.addLayer(result.randomVisualizer(), {}, 'clusters');


/////////////////////////////////////////
///////////// 5. Export ////////////////
///////////////////////////////////////


Export.image.toDrive({
  image: result,
  description: 'Classified Data',
  scale: 10,
  region: geometry
});
```

(Click here to see the code in GEE)[https://code.earthengine.google.com/bf602fc609a1f98f086b7f277bde8b27]
