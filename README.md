# Overview

This research uses satellite image data along with machine learning algorithms to predict the development of buildings in confined regions of interest around previous World Bank sites in Malawi. A random forest model performs best, achieving a 0.68 AUC-PR and 0.93 AUC-ROC on on validation data. The model was then applied to the 4 square kilometer and 25 square kilometer regions of interest around the World Bank sites for previous years. The model did not predict a higher building growth rate within more confined regions of interest than wider ones. Overall, this research approaches the problem of a lack of adequate data in low-resource settings, such as Malawi. It demonstrates a methodology for using publicly available data to predict the building development, a proxy for economic activity, of a confined region or interest.

# Research questions

1. Can machine learning tools be applied to publicly available satellite data in order to predict localized building development?
2. Can a trained machine learning model be applied to images of former World Bank project sites in order to predict the development of these sites in comparison to wider regions?
3. Does proximity to a World Bank project focusing on infrastructure and local economic development lead to greater development in the Malawian context?

# Data used and processes

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

This yielded labelled data sets with pixel information: coordinates, band data and building, which could be used to predict building based on the band values. The code for cleaning the pixel and buildings datasets and joining the data based on pixels (coordinate points) within buildins (coordinate polygons) is detailed in the script: **7_pixel_ob_cleaning_merging.ipynb** in this repository. The cleaned datasets were joined with code the in script: **8__labelled_data_joining.ipynb** The combined pixel and building data for each project ROI are stored in datasets in the folder **combined_pixel_ob** within the data folder this repository. 

Google earth Satellite image with Open Buildings Overlay and pixels

<img width="975" alt="image" src="https://user-images.githubusercontent.com/78730842/225679609-d9cccc10-d749-4885-80f0-b71986cecdb4.png">

This is the same zoom-in of Zomba, Malawi as above. Black dots represent pixels not part of a building. Blue dots represent pixels part of a building. The image shows that the classification of pixels based on their coordinates being inside of building polygon coordinates from the Open Buildings dataset was successful. 

# Data description

The combined pixel-building datasets for each of the ROIs were combined to make a training dataset consisting of 1,673,534 unique pixel observations from the 40 project sites in 2021, 69,258 were part of a building. This represents 4.138\% of all pixels. The figure, Average spectral band values by building label, below demonstrates the proportion of pixels labelled as part of a building (1) and not-building (0). As expected, and demonstrated, sites designated as urban have the highest proportion of pixels being labelled as building and semi-urban sites have a higher proportion of building pixels than rural sites. 

The figure, Spectral band value distributions, below demonstrates the average band values for pixels which are part of buildings and not part of buildings. As shown, the blue (B2), green (B3), red (B4) and short-wave infrared 1 (B12) values are all on average higher for pixels that are part of buildings as opposed to those not part of buildings. The near infrared (B8) and short-wave infrared 1 (B11) are on average the same for the different pixel categorizations. The following figure also shows the distribution of band values for pixels labelled building and non-building. The code for exploring the data can be found in the sript: **9_data_exploration.ipynb**.

Average spectral band values by building label

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/235947160-1d0e6962-bb0c-4517-8b7c-08f9722e6c0b.png>

Spectral band value distributions

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236177904-d8ffff05-0754-408c-935e-4e8f6f79b729.png>

Pixel band value distributions, darker shade for building  = 1, lighter shade for building = 0. The plots show that the distribution of values for each spectral band is relatively normally distributed. There is considerable overlap between the distribution of band values for pixels labelled building = 1 and building = 0, especially for B8 (NIR) and B11 (SWIR 1). But the values are higher for building = 1 for B2 (blue), B3 (green), B4 (red) and B12 (SWIR 2).

# Machine learning

The process for splitting the data can be found in the script: **10_data_splitting.ipynb**
And the models were trained with code in the script: **11_training_ml.ipynb**

**Explanation of algorithms**

***Naive Bayes***

The naive Bayes classifier simplifies learning by assuming that features are independent given classes. Although independence is generally a poor assumption, in practice naïve Bayes often performs well as classification error is not necessarily related to the quality of the fit to a probability distribution.

***Logistic regression***

Logistic regression is a maximum likelihood classification that, as a supervised machine learning algorithm developed for learning classification problems, maximizes the likelihood of classification.

***Extreme gradient boosting***

Extreme gradient boosting (XGBoost) is an implementation of gradient boosting, a technique that combines several weak models into a stronger ensemble model. The algorithm works by iteratively training a series of decision trees on the training data, where each subsequent tree tries to correct the errors of the previous tree. XGBoost minimizes the sum of the loss function and a regularization term, which helps to prevent over-fitting.

***Random forest***

Random forests (RF) are an ensemble learning method that use multiple decision trees. A decision tree is a machine learning algorithm that recursively partitions the feature space into smaller and smaller subsets based on the values of the input features. Each decision tree in the RF is trained on a random subset of the training data and a random subset of the features. The final prediction of the RF is made by aggregating the predictions of all the decision trees in the forest.

**Model training**

With the initial labelled dataset of 1,633,659 unique pixel observations, I trained models with the algorithms described above on a subsample of 80% of the dataset, the remaining 20% of observations were reserved for model testing. 

I was concerned that the highly imbalanced nature of the data, with positive observations only making up 4.223\% of total observations, would lead the trained models to underestimate positive predictions. Therefore, I balanced the data through over-sampling. This was achieved by replicating the minority class, building = 1. Another technique for balancing a dataset is under-sampling, which reduces the number of majority observations. But research has shown that models trained on over-sampled data perform better than those trained on under-sampled data. The over-sampled dataset consisted of 3,208,552 observations. As the XGBoost and RF models trained on the imbalanced data performed best, I trained them also on the over-sampled data.

I was also curious if models trained based on urbanity would better predict buildings in images of that same urbanity. I therefore split the initial labelled dataset into three urbanity datasets; urban, semi-urban and rural. I then over-sampled each of these. Based on the strong performance of the RF model trained on over-sampled data, I trained a RF model on each of the over-sampled urbanity datasets. In the end, I trained 10 classification models: 

1. Naïve Bayestrained on imbalanced data
2. Logistic regression trained on imbalanced data
3. KNN trained on imbalanced data
4. XGBoost trained on imbalanced data
5. RF trained on imbalanced data
6. XGBoost trained on over-sampled data
7. RF trained on over-sampled data
8. RF trained on over-sampled urban data
9. RF trained on over-sampled semi-urban data
10. RF trained on over-sampled rural data

# Findings

**Test evaluation**

I first tested the models that were trained on the imbalanced data. The test data, which was 20% of the original dataset, consisted of 334,707 observations, 13,673 (4.259%) were positive observations (building  = 1).  The table below shows four different evaluation metrics for each of the models.

Results table for text evaluations on imbalanced data
<img width="700" alt="test_eval_tb" src="https://user-images.githubusercontent.com/78730842/236181052-4c185450-9ea8-4092-bd0c-95e7f7e813b1.png">

As the data was highly imbalanced, I was concerned that the models would under-predict positive observations. This would lead to models that systematically underestimate the number of buildings in unseen images. Based on the results shown in the table above, this was the case. All of the models had high accuracy on the test data; naïve Bayes was the lowest with 0.92 and the rest had 0.96, but this high accuracy was likely to do with the data being highly imbalanced. The precision, recall and F1-scores for positive observations were very low. The fact that the precision scores were higher than the recall scores for all of the models, except naïve Bayes, shows that the instances of false negative were higher than the instances of false positives.

As mentioned above, I also evaluated the models based on the areas under the precision-recall and ROC curves. The results of evaluations can be seen in the figures below.

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236189923-421a0f0b-d53b-49df-b6e8-ffddcb6386be.png>

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236190049-b560ac78-25f0-464d-8f70-57ff0f1e601b.png>

Analysis of the areas under the curve reveal that XGBoost and RF performed the best, both in terms of PR and ROC. The AUC-PR and AUC-ROC were 0.34 and 0.90 for XGBoost; and 0.38 and 0.90 for RF. Although naïve Bayes had the highest positive recall, 0.31, its AUC-PR and AUC-ROC curves was significantly lower than for the XGBoost and RF models. This implies that XGBoost and RF were the best performing models trained on imbalanced data. But there was still a concern that due to their low positive recall scores; 0.10 for XGBoost and 0.15 for RF, they would systematically underestimate building counts.

Based on the comparatively good performance of the XGBoost and RF models on the imbalanced data, I used these algorithms to train models on the over-sampled data. As mentioned above, I also trained RF models based on urbanity. The table below shows evaluation metrics for each of the models. 

Results table for text evaluations on over-sampled data
<img width="700" alt="test_eval_tb_over" src="https://user-images.githubusercontent.com/78730842/236191299-6e924fc5-bfd2-4b56-938e-e691ab214009.png">

Having been trained on the over-sampled data, the RF models performed much better on predicting positive observations. The positive recall score for the over-sampled XGBoost model was 0.85 with the negative recall score being 0.80. This implies that that model would over-predict positive observations. But the RF trained on the over-sampled data and the RF models trained on the over-sampled urbanity datasets achieved very higher scores for all metrics, the lowest being 0.97 for the urban RF for positive precision and negative recall.  

As with the models trained on imbalanced data, I evaluated the models based on the areas under the precision-recall and ROC curves. The results of evaluations can be seen in the plots below. This reveals that the RF models performed better than XGBoost. The AUC-PR and AUC-ROC were 0.36 and 0.91 for XGBoost, only nominally better than the XGBoost trained on the imbalanced data. This implies that, although the precision and recall were higher for the XGBoost over-sampled model for positive observations, the lower precision and recall for the negative observations in the over-sampled XGBoost as compared to the imbalanced XGBoost, hurt the over-sampled model’s performance. All of the over-sampled RF models scored 1.00 for the AUC-ROC. The over-sampled RF also scored 1.00 for the AUC-PR. The urbanity over-sampled RF models were tested with observations from the same urbanity. The urban RF model scored the best with 0.89 AUC-PR, then 0.46 for the semi-urban RF and 0.18 for the rural RF. The code for conducting the test evaluation can be found in the script: **12_test_evaluation.ipynb**.

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236191661-db3821f3-45a0-4c4b-b98d-cc639f3a5b58.png>

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236191754-1301ad97-9ca7-4785-9f17-0a85e6cdce03.png>

**Validation evaluation**

Although the training datasets were large; 1,338,827 observations for the imbalanced and 2,566,841 observations for the over-sampled training dataset, I was concerned that there would be certain unique patterns in the data based on these observations coming from only 40 unique satellite images; one image from each of the project sites of interest in 2021. Patterns in the data could make the models over-fit, which would limit their generalizability. 

Therefore, I constructed a validation dataset, which consisted of pixel data from images of sites that were included in the AidData geocoded dataset, but not projects of interest and therefore not included in the training and testing data. These sites were not included in the predictive analysis as they were from projects that would not arguably spur localized economic development in a way that would be observable through satellite imagery. This validation dataset consisted of 1,035,561 pixel observations from satellite 26 images. 

The models that performed the best on the validation data were the RF trained on the imbalanced data, the XGBoost trained on the over-sampled data, the RF trained on the over-sampled data and the RF trained on urban over-sampled data, as shown in the table below. The recall values for these models, except XGBoost, are higher for negative observations than positive observations implying the models are underestimating the number of positive observations. 

<img width="700" alt="val_eval_tb" src="https://user-images.githubusercontent.com/78730842/236192622-b14dee30-b7a5-4c4b-8df4-768c623444b6.png">

The plots below show the PR and ROC curves for the models on the validation data. The AUC-PR for the RF models are all similar, with RF over-sampling 0.68, then RF urban over-sampling 0.64 and RF from the imbalanced data at 0.60. The AUC-ROC for these models are also similar, with 0.93 for the RF over-sampling and 0.92 for the others. The XGBoost performance is lower, 0.26 AUC-PR and 0.89 AUC-ROC are much lower. I found it interesting how, for the RF models, the precision score stays at almost 1.0 until recall reaches around 0.5, then the precision scores drop rapidly. The code for conducting the model validation can be found in the script: **13_validation_evaluation.ipynb**.

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236192805-ccf713d8-3d69-46a7-b744-95b388bb9bcb.png>

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236192927-ced44b5b-515b-42f5-abbf-076fc0ad2d2a.png>

# Model application and analysis

My second and third research questions were: Can a trained machine learning model be applied to images of former World Bank project sites in order to predict the development of these sites in comparison to wider regions? and; Does proximity to a World Bank project focusing on infrastructure and local economic development lead to greater development in the Malawian context? In order to address these questions, I had to apply the chosen machine learning model to satellite image data from the same sites for previous years and to data from images of wider regions. I downloaded the pixel data from the same World Bank sites as used in the training data for the years before 2021, images from 2016 – 2020. I then applied the best-performing model, the RF trained on over-sampled data, to this data in order to predict the building development of these sights over this time period. The code for applying the model can be found in the script: **14_ml_application.ipynb**.

To address the proximity question, I downloaded pixel data from Sentinel-2 images from around the same project sites, but a rectangle of 25 square kilometers as opposed to the 4 square kilometer regions of interest I had analysed up to this point. For the 4 square kilometer ROIs, the average growth rate per year were: 2017 (0.093), 2018 (0.186), 2019 (0.076), 2020 (0.775) and 2021 (2.561). The average building growth rate per year was 0.577. For the 25 square kilometer ROIs, the average growth rate per year were: 2017 (0.034), 2018 (0.046), 2019 (0.188), 2020 (0.306) and 2021 (2.906). The average building growth rate per year was 0.634. The average annual growth rate for the 25 square kilometer regions of interest was therefore slightly higher than for the 4 square kilometer regions of interest around the project sites. This implies that the predicted building growth rate for areas in closer proximity to the project sites were actually slightly less than in the wider proximity. The code for analysing the predictions made by the model can be found in the script: **15_analysis.ipynb**.

<img width="500" alt="image" src=https://user-images.githubusercontent.com/78730842/236193922-fa9dde22-60b7-42b9-9d94-a3e5b0d1ddb3.png>
