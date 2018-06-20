# Landsat_sentinel_fusion
Just cast two kinds of settilite images to one image.

// STEP 1:
// Set a Point Spread Function (PSF) with size of kernel and size of pixel:

var size_kernel = function(coarse_resolution, fine_resolution) {
  return coarse_resolution/fine_resolution;
};

var PSF = function(N, size) {
  var weights = 1/size;
  var list = ee.List.repeat(weights, N);
  var box = ee.List.repeat(list, N);
  var kernel = ee.Kernel.fixed({
    width: N,
    height: N,
    weights: box,
    normalize: true
  });
  return kernel;
};

// STEP 2:
// Convolution operation:

var convolution = function(image, PSF) {
  return image.convolve(PSF);
};

var Zc = function(collection, convolution) {
  return collection.map(convolution)
}

// STEP 3:
// Linear regression:

var trend = function(collection, band_independent, band_dependent) {
  var bandlist = collection.select([band_independent, band_dependent]);
  var linearRegression = bandlist.reduce(
  ee.Reducer.linearRegression({
    numX: 1,
    numY: 1
  }));
  var coefficients = linearRegression
  .select(['coefficients'])
  .arrayFlatten(band_independent);
};

// Step 4:
// finding residual:

var residual = function(a, b, Z1, Z2) {
  return (Z2 -(a*Z1+b));
};
