# Detecting Car Camping with Object Classification

---
This project aims to increase awareness and access to public lands, of which the most approachable use is car camping, also known as dispersed camping or RVing.

The scope focuses on a single county in Colorado, with a high density of public land, and where camping is permitted on any of numerous forest roads.


<br><br>

For a more clear walkthrough of the project, visit my medium account, where I'm breaking the workflow into a 4 part blog series.

[Part 1: Data Acquisition & Pre-Processing](https://medium.com/@kendallfrimodig/efficient-object-detection-within-satellite-imagery-using-python-85331d71ff69)


<br><br>



### Summary

---

- US Department of Agriculture orthoimagery was clipped by a region of interest, a buffer of 200 feet from any national forest, BLM, or state trust road. 

- THe imagery was then reduced via a moving window raster slicer, and only pertinent training and testing images were retained.

- Bounding boxes were manually assigned and converted to a mask, for 180 campsites at known destinations across the county.

- The Test data consisted of the remaining (1900) image tiles of candidate roads for dispersed camping

- Object detection training, and geo-referenced predictions for unseen data are planned to be output using a Mask-R-CNN.

- Due to high computation time of an object localization problem, and a lack of a baseline model for the question at hand, the model is still under development


### Files


---
<br>



[ee_api_colab.ipynb](./notebooks/ee_api_colab.ipynb):

    - Co-Lab notebook (not to be ran locally) which reads, transforms, and exports data from Google Earth Engine

[preprocessing-eda.ipynb](./notebooks/preprocessing-eda.ipynb):

    - Local notebook for converting source imagery into model-friendly micro tiles

[vizualization.ipynb](./notebooks/vizualization.ipynb):

    - Displays GIS data, giving context for the projects initial scope and pre-work in qgis

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

**due to the tile and mask folders equaling 40GB in total, only the sub-tiles containing data for training were uploaded (~90)**

---

## Background

---

With the booming populations of Western US cities such as Denver, Seattle, or Salt Lake City, comes
an increase in the outdoor recreation traffic. A relatively narrow range of free camping
destinations results in congestion at popular sites and even some having to drive deep into a road or
turn around. This also concentrates the human imapact on the forest to these few areas,
scavenging for firewood leaves forest floor bare often and people often take wood
directly from trees. Increasing the range of camping could help manage fuels for potential wildfires.

The US Forest Service (USFS) maintains maps with manual designations for how public roads are to be used, each
national forest adopting their own policy and not releasing such data within the recreational roads data containing
90 other features of the roads. These Forest Roads are in most cases former logging sites, on the perimeters of more
conserved areas such as Wilderness or National Parks.

Until very recently, there was no option for easily viewing which public land roads were designated
for camping.

For many years the best resource for finding public camping areas was 'Free-Camping.com' which is a
croudsourced effort, and only scratches the surface of public land access.

Meanwhile Freeroam, integrates the Motor Vehicle Use Maps (MVUM), but UI is lacking and no granular
context regarding the quality of road, where the sites actually are, seasonality
and extremly difficult to transpose the information to navigation apps.

Additional issues with sites such as 'Free-Camping'

- Usually there is no designation for where precicesly camping is permitted along a road
- Often times can be private before and after
- Larger areas have many branches of roads, which would be worth taking, and what ones might get you
into trouble (with a regular car)










# Methods


After realizing no publicly available sattelite imagery would be suitable for the project, I settled on the US Department of Agricultures NAIP imagery which is taken via plane

First tried to use NAIP JPEG files from USGS National Map

- Lower resolution, data only available for 2015, though this data is collected every two years.


Next sourced the USDA source files
- Mosaic, or collection of images in original uncompressed format
- Each state is covered every other year, so 2020 data was available.
- Comparing 2015 to 2020, there were noticeably more campsites present after 5 years

Issues
- Image was in '.sid' format
- Couldn't perform any operations on this format
- Once uncompressed, was 40gb

Though the file was 40gb, it should be much smaller after I selected only pixels within a few hundred feet of forest, blm, or state trust roads. After clipping 90% or more of the pixels, the file was still 36gb in size.

Using JPEG compression, the size was reduced but I wanted to automate the whole workflow so the project could scale, so I moved back to the cloud.

Used the Earth Engine api for aquiring and clipping image collection

- Earth Engine has a native app, but is only for javascript
- A python library is available, that has the same functions but has to be performed in a notebook outside the earth engine console

Earth Engine itself has a storage quota, so when performing large scale analysis its best to export results to google cloud storage or google drive. However this requires separate libraries, and authentication processes beyond the earth engine authentication (inside colab or local notebook). Additionally, a google cloud project must be set up to 'talk' with the earth engine. This is a convuluted process and there are many different methods of setting this up in google cloud / notebook. Googles documentation is primarly in javascript, so it's difficult to find reliable information with regards to setting the platforms up in python

- Using a rectangle, and its coordinates, I queried the earth engine to return all the NAIP images located in the bounds

- With this list, I looped through each image, clipping the pixels by the buffer of selected roads, and exporting the result to google cloud storage.

- If there were no roads within that image, it would take some time to process but the output would result in only a black image of 3mb.

Each individual image that contained pixels of interest, was considerably smaller than the local results, only being 27mb each. The quality and resolution of the pixels was investigated and comparable to the source data. Each image contained 4 bands, RGB and Near Infared, for the pixels within a few hundred feet of a public road.


## Modeling



Evaluation metrics for the Mask-R-CNN are evaluated on the Intersection of Union (IoU) between the predicted mask and the ground truth. This is essentially the percent overlap between the reference bounding boxes and predicted object boundary. A True detection is considered where the predicted boundary has at least 50% overlap with the true boundary

Since the data were manually annotated with bounding boxes, the primary metric considered would not be utilized yet. In order to reliably produce this metric, and ultimately improve model performence, each bounding box would have to be converted into a more detailed polygon representing the precise footprint of the site.

Precision and Recall can be derived from the model output by calculating the confusion matrix values, after calculating overlap for each object.

From <https://www.frontiersin.org/articles/10.3389/frai.2020.534696/full>

Since bounding boxes were not set for all possiple camping locations, and to maximize the labeled data for learning, the performence would be retroactively calculated by taking the predicted bounding boxes, and manually creating a best estimate polygon, calculating the IOU value finally. This would be somewhat subjective because sites can be obscured if not completley hidden by forest canopy, and manually creating a bounding polygon would not be considered an objective reference boundary.



# Conclusion




### Next Steps
Though the Cloud Platform and Earth Engine haven't been fully leveraged yet, heres some benefits for future directions.

Google earth engine has a function which takes an image and outputs a numpy array suitable for modeling. The other option is to use rasterio (built on top of the GDAL library) in a local instance or in colab. However GDAL is not a native python library and must be installed at a system level. For that reason a special python environment is needed for leveraging rasterio, and this makes things less nimble at scale. For this reason, I hoped to reload the images back from GCS and converted to a numpy array, and stored in an accompanying folder. My intial attempt was unsuccessful as the limit the pixel size for array extraction.

If I were to introduce my moving window image segmentation process into the co-lab notebook, then the arrays could be derived with the earth engine function.

This is more efficient since translating the entire mosaic as one image, into an array would amount to 50+ billion values in the case of Chaffee County, Colorado, since in order to retain georeference, the matrix needs to represent the nodata-pixels.

Performing the model on each individual image is superior since images containing only no-data can be skipped. Thus for the whole county, the combined size of the individual image arrays would be significantly less than an array of the entire composite, while still retaining the georeference for re-projecting the model output.



Having the individual images as cloud optimized GeoTIFFs will come in handy when creating an app, which displays the results of identified campsites as the entire image won't need to be loaded at once. Additionally images with areas of interest need to be rended on top of a basemap

The convenient inclusion  of the near infared band in Googles version of the NAIP data gives context for shadows, if added to an RGB composite it can differentiate the ground which would appear as black in the RGB data.

Since campsites are usually on level ground, including a 5th dimension in the array was considered, being the average slope at that pixel location. This would be possible by taking a DEM raster of the same size of the imagery, and calculating slope based on the neighboring elevation pixels.However if the elevation raster doesn’t line up perfectly, things are a slightly more complicated.

This could be done by using the imagery as an input grid, and calculate the blank raster using elevation point data. The point data could be sampled from a DEM, or 3DEP point cloud data which is very fine grained, but contains points for treetops in addition to the ground, so the 3DEP data would have to be filtered to just the ground point classification.

### Long Term Goals Revised

Overall, the original scope was limited to roughly 2% of the eventual area of interest chosen. Scalability was prioritized, and a great deal of time spent in scripting the processes in
google colab, where the external API's of cloud storage and the earth engine were leveraged for pre-processing and export.

Due to time contraints the efforts on the cloud front were put on hold, but I beleive to have a good start on what can eventually be a scalable web map

Original 'Stretch' Objectives

- Online application that shows polygons for sites, color coded roads for quality or vehicle required, and automatic masking of sites that are outside seasonal designated open/close
- Take one district from each national forest, create 300 foot buffer (rule) along Dispersed camping designated roads.
- Identify boundaries for pull offs - labeled as such in training data.
- Generate prediction boundaries for sites, using subset of roads. If model works then apply to adjacent ranger districts
