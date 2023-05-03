# Overview

This research uses satellite image data along with machine learning algorithms to predict the development of buildings in confined regions of interest around previous World Bank sites in Malawi. A random forest model performs best, achieving a 0.68 AUC-PR and 0.93 AUC-ROC on on validation data. The model was then applied to the 4 square kilometer and 25 square kilometer regions of interest around the World Bank sites for previous years. The model did not predict a higher building growth rate within more confined regions of interest than wider ones. Overall, this research approaches the problem of a lack of adequate data in low-resource settings, such as Malawi. It demonstrates a methodology for using publicly available data to predict the building development, a proxy for economic activity, of a confined region or interest.

# Research questions

1. Can machine learning tools be applied to publicly available satellite data in order to predict localized building development?
2. Can a trained machine learning model be applied to images of former World Bank project sites in order to predict the development of these sites in comparison to wider regions?
3. Does proximity to a World Bank project focusing on infrastructure and local economic development lead to greater development in the Malawian context?

# Process for data extraction and cleaning

**AidData geocoded World Bank projects** 

In order to gain data on the location of development projects, I started with the AidData dataset of geocoded World Bank projects from 1995 to 2014. This dataset consisted 137,572 of geolocated sites, which were part of 5,881 of World Bank projects in 151 countries. In addition to geocoded information (latitude and longitude points), the dataset consisted of approximately 200 variables, including the start and end dates of the projects, the World Bank team lead on the project, the project sectors and goals, financial disbursement details, and the Independent Evaluation Group (IEG) evaluations of the projects. 

As I was only interested in World Bank sites taking place in Malawi, I filtered for only those projects in that country. This consisted of 58 projects. Details on how the AidData data was joined and cleaned is in the script: **1_aiddata_world_bank_geo.Rmd** in this repository.

**World Bank projects in Malawi**

I then downloaded and cleaned data on all World Bank projects from the World Bank website. This dataset included information on the total project costs and descriptions of the project development objectives. Details on how the AidData data was joined and cleaned is in the script: **2_world_bank_malawi_cleaning.Rmd** in this repository.

**Joining World Bank Malawi datasets**

I joined the cleaned AidData dataset with the World Bank Malawi data. Joining this data resulted in a dataset of geocoded World Bank projects in Malawi between 1995 and 2014 with information on start dates, end dates, sectors, development objectives, IEG evaluation, and the project costs.

I filtered out invalid points and duplicated coordinates for the same projects. I also reduced the dataset to only projects that were locally focused (not focusing at central government level reform) and that could arguably generate economic expansion, such as infrastructure and social capacity building projects. This resulted in the dataset consisting of 5 unique projects: 
1. Road Maintenance Rehabilitation Project (P001666);
2. Social Action Fund; Infrastructure Services (Social Action Fund Project) (001668);
3. Infrastructure Services (P057761);
4. Community-Based Rural Land Development Project (P075247);
5. Irrigation, Rural Livelihoods and Agricultural Development Project (P084148);
6. Community Based Rural Development Project (P115226)

There were 85 geocoded locations in total from these projects. Removing duplicates reduced this to 40 unique project sites. Finally, I added a variable which identified the four geocoded points of a 4 square kilometer rectangle with the original geocoded point as the center. This is to have a region of interest in order to study building development within proximity to the geocoded project site.

The resulting dataset consisted of 85 sites within 5 World Bank projects along with the following variables for each project site:
-	Project name
-	Project description/objective
-	Project Start date
-	Project end date
- project sector
-	Project cost
- IEG evaluation
-	Geocoded point (latitude and longitude)
-	Polygon coordinates of a 1 square kilometer rectangle around the point

Details on how the AidData data was joined and cleaned is in the script: **3_malawi_wb_geolocated.Rmd** in this repository.

**Open Buildings data**

Open Buildings was an initiative for which experiments were carried out using a dataset of 100k satellite images across Africa containing 1.75M manually labelled building instances, and further datasets for pre-training and self-training. The Open Buildings dataset consists of 516M Africa-wide detected buildings (Sirko et al. 2021). The data consists of polygon coordinates for identified buildings. These buildings were assumed to be present at the time of this research, 2021. From the Open Buildings database, I downloaded the data on all identified buildings in Malawi. Details on how the Open Buildings data was joined and cleaned for data on buildings in Malawi is in the script: **4_open_buildings_cleaning_malawi.ipynb** in this repository. The data for buildings in Malawi was saved in Google Drive as open_buildings_malawi.csv.

Using the coordinates of the 1 square kilometer around the project locations, I extracted data on the location of buildings (in 2021) for each 4 square kilometer region of interest. I only used data for regions of interest with at least 200 identified buildings. Details on how the Open Buildings data was reduced for the region of interest (roi) around each World Bank project site is in the script: **5_open_buildings_projects_sites_malawi_roi.ipynb** in this repository. The Open Buildings data for buildings in each project ROI are stored in datasets in the folder **open_buildings** in this repository.

Google Earth Satellite image with Open Buildings Overlay
<img width="974" alt="image" src="https://user-images.githubusercontent.com/78730842/225680043-384b97e6-962b-43e9-9dee-105cc8de0e20.png">
This is a zoomed in image of the city of Zomba, Malawi.
Green represents buildings identified with over 70\% confidence.
Yellow represents buildings identified with 65 - 70\% confidence.
Red represents buildings identified with 60 - 65\% confidence. 
The image shows that the predictive model development through 
Open Buildings was very successful at predicting building locations
in Africa. 

**Pixel data**

Using the Google Earth Engine API, I then downloaded the pixel data for August 2021 Sentinel-2 satellite images for each of the corresponding regions of interest. August was chosen because it is the least cloudy month in Malawi and 2021 because this was the year that buildings were identified by the Open Buildings research. Sentinel-2 images have 10-meter resolution (each pixel represents 10 square meters). Sentinel-2 images were chosen over other publicly accessible images, such as Landsat-8 or Landsat-9, because these satellites produce 30-meter resolution images. Each of the images of the regions of interest consist of approximately 40,000 pixels. The code for downloading the pixel data for an individual ROI is detailed in the script: **6_pixel_data_extraction_ee** in this repository. Keep in mind that this is Java Script to be used within the Google Earth Engine Code Editor. For information on how to register for the Earth Engine API and access data through the Code Editor, visit: https://earthengine.google.com/platform/.

Data for each pixel within the images consisted of the band values for that pixel (red, green, blue, infrared, short-wave infrared 1 and short-wave infrared 2) along with the coordinates for each pixel. The pixel data for each project ROI are stored in datasets in the folder **ee_pixel_data** in this repository.

**Joining pixels with buildings**

I therefore had the buildings data, which gave the polygon coordinates or each building, in the regions of interest and the pixel data for a Sentinel-2 image for those ROIs in August 2021. I then joined the datasets based on the coordinate points for the pixels being within the polygon coordinates, using the gds.sjoin function from the GeoPandas library in Python. This identified each pixel within each Sentinel-2 image (August 2021) as being part of a building or not part of a building. I created a new variable based on this information: “building”. Pixels that were part of a building were labeled building = 1, and those not part of a building were labeled building = 0. 

This yielded labelled data sets with pixel information: coordinates, band data and building, which could be used to predict building based on the band values. The code for cleaning the pixel and buildings datasets and joining the data based on pixels (coordinate points) within buildins (coordinate polygons) is detailed in the script: **7_pixel_ob_cleaning_merging.ipynb** in this repository. The combined pixel and building data for each project ROI are stored in datasets in the folder **combined_pixel_ob** in this repository. 

Google earth Satellite image with Open Buildings Overlay and pixels
<img width="975" alt="image" src="https://user-images.githubusercontent.com/78730842/225679609-d9cccc10-d749-4885-80f0-b71986cecdb4.png">
This is the same zoom-in of Zomba, Malawi as above.
Black dots represent pixels not part of a building.
Blue dots represent pixels part of a building.
The image shows that the classification of pixels based on their coordinates
being inside of building polygon coordinates from the Open Buildings dataset was successful. 

# Data description

The combined pixel-building datasets for each of the ROIs were combined to make a training dataset consisting of 1,673,534 unique pixel observations from the 40 project sites in 2021, 69,258 were part of a building. This represents 4.138\% of all pixels. Figure~\ref{fig:prop_build_pix} below demonstrates the proportion of pixels labelled as part of a building (1) and not-building (0). As expected, and demonstrated in \ref{fig:prop_pix_urbanity}, sites designated as urban have the highest proportion of pixels being labelled as building and semi-urban sites have a higher proportion of building pixels than rural sites. 

The figure below demonstrates the average band values for pixels which are part of buildings and not part of buildings. As shown, the blue (B2), green (B3), red (B4) and short-wave infrared 1 (B12) values are all on average higher for pixels that are part of buildings as opposed to those not part of buildings. The near infrared (B8) and short-wave infrared 1 (B11) are on average the same for the different pixel categorizations. The following figure also shows the distribution of band values for pixels labelled building and non-building. 

Average spectral band values by building label
<img width="975" alt="image" src=https://user-images.githubusercontent.com/78730842/235947160-1d0e6962-bb0c-4517-8b7c-08f9722e6c0b.png>


**Machine learning**

Labeled variable (y): building (1/0)
Feature names (X): bands values (red, green, blue, infrared, short-wave infrared 1 and short-wave infrared 2)

Train and test various machine learning categorization methods – RF, BoostedDT, SVM, KNN, neural net (?)

Download the pixel band data for Sentinel-2 images for the same regions of interest for 2017, 2018, 2019, 2020 and 2022.

Once a proper ML tool is chosen for predicting pixels as being building or not building for each region of interest (for August 2021), apply it to the images from that same region or interest for the other years 2017, 2018, 2019, 2020 and 2022 in order to predict which pixels are buildings (and not buildings). Compare the predicted presence of buildings in the images at each region of interest to estimate the development of buildings over time at the World Bank project sites. 
