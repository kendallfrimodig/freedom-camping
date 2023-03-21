# Detecting Car Camping with Object Classification

---



The overall aim of this project is to streamline the pre-processing of satellite imagery for object recognition modeling. 

<br> 

**The code takes two inputs from a GIS front end**

<br> 

- clipped version of the satellite imagery via a shapefile which defines where training and test images can be found

<br> 

 - rasterized collection of bounding boxes for the object of interest - either drawn manually or from pre-existing data. 

<br> <br> 


All steps for pre-processing the data for an object recognition model are then automated.

<br><br>

Once the inputs are created, the imagery collection and bounding box raster are then reduced to a smaller set of model compatible tiles where objects exist, or predictions could be made. Text annotations are then created for each tile containing a labeled object, by analyzing its matching mask tile. Then, images are sorted into train or test folders. 


<br><br>

### Background

This project aims to increase awareness and access to public lands, of which the most approachable use is car camping, also known as dispersed camping or RVing. The scope focuses on a single county in Colorado, with a high density of public land, and where camping is permitted on any of the numerous forest roads. Locations for dispersed camping are not readily provided by all USFS offices. When this information is provided, it's usually in paper maps and difficult to source or understand for the average camper. 


<br>

For a more clear walkthrough of the project, visit my medium account, where I'm breaking the workflow into a 4 part blog series.

[Part 1: Data Acquisition & Pre-Processing](https://medium.com/@kendallfrimodig/efficient-object-detection-within-satellite-imagery-using-python-85331d71ff69)




<br><br>

### Summary

---

- US Department of Agriculture orthoimagery was clipped by a region of interest, a buffer of 200 feet from any national forest, BLM, or state trust road. 

- The imagery was then reduced via a moving window raster slicer, and only pertinent training and testing images were retained.

- Bounding boxes were manually drawn in QGIS and converted to a raster file where pixels indicate box locations, for 180 campsites at known destinations across the county.

- The Test data consisted of the remaining (1900) image tiles of candidate roads for dispersed camping

- Predictions for unseen data are planned to be output using a object recognition model.

- Due to limited hardware capabilities and issues transferring all assets to the cloud for modeling in co-lab, the model is still under development


<br><br>

### Files


---
<br>

Notebooks

[ee_api_colab.ipynb](./notebooks/ee_api_colab.ipynb):

- Co-Lab notebook (not to be ran locally) which reads, transforms, and exports data from Google Earth Engine
- (not ultimately utilized but retained for anyone interested in seeing this method)

[preprocessing-eda.ipynb](./notebooks/preprocessing.ipynb):

- Local notebook for converting source imagery into model-friendly micro tiles

[vizualization.ipynb](./notebooks/vizualization.ipynb):

- Displays GIS data, giving context for the projects initial scope and pre-work in qgis


[Co-Lab Model](https://colab.research.google.com/drive/1kn_0hb6SgXVEqNms4JfLIzQUtOCYo9IA?usp=sharing)

- Notebook for applying model

<br>

Data

- Tiles: output of pre-processing notebook, totaling 12,000 sub tiles of original data
- Masks: same as tiles with a matching file naming convention for each image slice

* [aoi.GeoJSON](./data/Polygon/aoi.GeoJSON): Buffer used to clip image composite, leaving pixles in buffer
* [labels.GeoJSON](./data/Polygon/labels.GeoJSON): Bounding boxes for training data mask
* [campspots.csv](./data/Point/campspots.csv): Known locations of camping areas (freecamping.com)
* [mergedroads.GeoJSON](./data/Line/mergedroads.GeoJSON): Combination of USFS, BLM, and complete 'TRAN' files to extend AOI
* [USFSRoads.GeoJSON](./data/Line/USFSRoads.GeoJSON): Only forest service roads
* [chafee_clipped_compressed.tif](./data/chafee_clipped_compressed.tif): Composite of 2019 <1m resolution imagery, converted from .sid and compressed from 40gb to 1gb
* [masked.tif](./data/masked.tif): Output of 'rasterize' function with bounding boxes as input

<br>

QGIS Projects
* [labeling.qgz](./gis/labeling.qgz): Creation of bounding boxes over area of interest imagery
* [preprocessing.qgz](./gis/preprocessing.qgz): Creation of merged public road datasets and buffer for AOI

<br>

**due to the tile and mask folders equaling 40GB in total, only the tiles containing data for training were uploaded (~90)**

---

