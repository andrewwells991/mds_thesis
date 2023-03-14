# mds_thesis

Process for data extraction and cleaning

In order to gain data on the location of development projects, I started with the AidData dataset of geocoded World Bank projects from 1995 to 2014. This dataset consisted 137,572 of geolocated sites, which were part of 5,881 of World Bank projects in 151 countries. In addition to geocoded information (latitude and longitude points), the dataset consisted of approximately 200 variables, including the start and end dates of the projects, the World Bank team lead on the project, the project sectors and goals, financial disbursement details, and the Independent Evaluation Group (IEG) evaluations of the projects. Details on how AidData data was joined and cleaned is in the script: 1_aiddata_world_bank_geo.Rmd in this repository.

As I was only interested in World Bank sites taking place in Malawi, I filtered for only those projects in that country. This consisted of 58 projects.

I joined this geocoded data with the data on projects in Malawi from the World Bank website. This dataset included information on the total project costs and descriptions of the project development objectives. Joining this data resulted in a dataset of geocoded World Bank projects in Malawi between 1995 and 2014 with information on start dates, end dates, sectors, development objectives, IEG evaluation, and the project costs.

I filtered out invalid points and duplicated coordinates for the same projects. I also reduced the dataset to only projects that were locally focused (not focusing at central government level reform) and that could arguably generate economic expansion, such as infrastructure and social capacity building projects. This resulted in the dataset consisting of 5 unique projects: 
1.	Road Maintenance Rehabilitation Project (P001666); 
2.	Social Action Fund; Infrastructure Services (P001668); 
3.	Community-Based Rural Land Development Project (P075247); 
4.	Irrigation, Rural Livelihoods and Agricultural Development Project (P084148); 
5.	Community Based Rural Development Project (P115226)

There were 85 geocoded locations in total from these projects. Finally, I added a variable which identified the four geocoded points of a 1 square kilometer rectangle with the original geocoded point as the center. This is to have a region of interest in order to study building development within proximity to the geocoded project site.

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

Open Buildings was an initiative for which experiments were carried out using a dataset of 100k satellite images across Africa containing 1.75M manually labelled building instances, and further datasets for pre-training and self-training. Novel methods for improving performance of building detection with this type of model, including the use of mixup (mAP +0.12) and self-training with soft KL loss (mAP +0.06). The resulting pipeline obtains good results even on a wide variety of challenging rural and urban contexts, and was used to create the Open Buildings dataset of 516M Africa-wide detected footprints (Sirko et al. 2021). The polygon coordinates for the buildings are identified. These buildings were assumed to be present at the time of this research, 2021. From the Open Buildings database, I downloaded the data on all identified buildings in Malawi. 

Using the coordinates of the 1 square kilometer around the project locations, I extracted data on the location of buildings (in 2021) for each 1 square kilometer region of interest. I only used data for regions of interest with at least 200 identified buildings. 

Using the Google Earth Engine API, I then downloaded the pixel data for August 2021 Sentinel-2 satellite images for each of the corresponding regions of interest. August was chosen because it is the least cloudy month in Malawi and 2021 because this was the year that buildings were identified by the Open Buildings research. Sentinel-2 images have 10-meter resolution (each pixel represents 10 square meters). Sentinel-2 images were chosen over other publicly accessible images, such as Landsat-8 or 9, because these satellites produce 30-meter resolution images. Each of the images of the regions of interest consist of approximately 20,000 pixels. 

Data for each pixel within the images consisted on the band values for that pixel (red, green, blue, infrared, short-wave infrared 1 and short-wave infrared 2) along with the coordinates for each pixel. 

I therefore had the buildings data, which gave the polygon coordinates or each building, in the regions of interest and the pixel data for a Sentinel-2 image for those regions of interest in August 2021. I then joined the datasets based on the coordinate points for the pixels being within the polygon coordinates. This identified each pixel within each Sentinel-2 image (August 2021) as being part of a building or not part of a building. I created a new variable based on this information: “building”. Pixels that were part of a building were labeled building = 1, and those not part of a building were labeled building = 0. 

This yielded labelled data sets with pixel information: coordinates, band data and building, which could be used to predict building based on the band values.

Labeled variable (y): building (1/0)
Feature names (X): bands values (red, green, blue, infrared, short-wave infrared 1 and short-wave infrared 2)

Train and test various machine learning categorization methods – RF, BoostedDT, SVM, KNN, neural net (?)

Download the pixel band data for Sentinel-2 images for the same regions of interest for 2017, 2018, 2019, 2020 and 2022.

Once a proper ML tool is chosen for predicting pixels as being building or not building for each region of interest (for August 2021), apply it to the images from that same region or interest for the other years 2017, 2018, 2019, 2020 and 2022 in order to predict which pixels are buildings (and not buildings). Compare the predicted presence of buildings in the images at each region of interest to estimate the development of buildings over time at the World Bank project sites. 
