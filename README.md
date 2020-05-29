# Predicting number of days until a Department of Buildings Complaint is inspected

#### Authors:
- __Jeremy Ondov:__ [GitHub](https://github.com/ondovj)
- __Mahdi Shadkam-Farrokhi:__ [GitHub](https://github.com/Shaddyjr) 

---
## Problem Statement
In New York City, building violations are frequently reported to the Department of Buildings (DOB), which is often initiated by a citizen. However, the length of time between the report and a response by the DOB may vary greatly depending on the location, violation type, and other factors. Citizens expectations for when a complaint will realistically be inspected would allow them to make more informed decisions, such as whether or not to inquire about their ticket or resend a duplicate complaint. The Department of Buildings would like-wise benefit from the reduction in redundant complaints. To this end, __we will create a predictive model that can estimate the number of day until a complaint is inspected.__

Given the continuous nature of the target, we will explore various __regression models and will use R^2 as our metric__ for model selection.

## Executive Summary

We gathered data from the Department of Buildings(DOB) on the Open NYC platform around September 2019. These data were mostly categorical and few in number, which proved challenging to work with. Our target variable was also something we would have to construct based on the time a complaint was filed into record and when it was actually inspected. Right away we saw some __odd bevaior - most notably some observations had a negative number of days until inspection,__ meaning some complaints were filed after they were inspected. We later discovered this was due to some on-site inspections leading to staff-filed complaints, which were immediately inspected and filed later. We removed these unusual instances from the dataset.

Another issue we ran into was dealing with the sheer size of the dataset. We could not possibly load years of data into local memory, so we opted to sample at regular intervals from our dataset. This invariably leads to an assumption that the observations we sampled were representative of the entire dataset. For the most part, the rest of the cleaning process was fairly standard. 

After some preliminary modeling, we found our features to be lacking at predicting the target. So, we pulled additional data we felt may play a significant roll in predicting out target and merged the two. The outside data was census data, specifically focused on median income by zip code. Of course, this adds another dimension of complexity, since __we are almost certianly including implicit features associated with zip code, such as demographic and social economical confounders__ we may not want to include within our models. We also __conducted transfer learning by using K-Modes to engineer a new feature,__ representing each observation's most closely related cluster.

We then took basic preliminary steps to explore the data, such as looking for trends within the features that may have correlation to our target. We found a number of extreme outliers, suggesting some complaints have taken over 10 years to be inspected! Interestingly, __our target follows the gamma distribution,__ since it's related to "time until an event". We immediately considered using a GLM model. Unfortunately, due to a number of technical issues with the `statsmodels` package, we were unable to reach a satisfactory result. We then attempted a number of regression models, the best of which was a random forest, however it still performed poorly, considering the baseline. Lastly, we attempted to use time-series models, which also proved to be underfit.

__In the end, we were unable to find a satisfactory predictive model, though we may be able to gleem some information from interpretating our best model, a random forest.__


## Data
The dataset used in this project is available from NYC Open Data at https://data.cityofnewyork.us/Housing-Development/DOB-Complaints-Received/eabe-havv. This database compiles all complaints entered in the city from 1989, and is updated daily. For the purposes of this project, the dataset used was downloaded on or around September 23, 2019 and had complaints entered until September 21, 2019. Ideally, this tool would have constant access to the live database for the purposes of updating and retraining the predictive model to stay relevant.

## Data Dictionary
These data were cleaned from the original source using [the assocated cleaning notebook](./code/dob_data_cleaning.ipynb)

|Column | Data Type | Description|
|- | - | :-|
|special_district|string|Indicates whether or not the building identified in the complaint is located in a Special District.|
|complaint_category|string|DOB Complaint Category Codes (01-Accident Construction/Plumbing, etc.)|
|unit|string|The most recent unit that was assigned to this complaint. It may have been initially assigned to one unit, and then referred to another unit for disposition.|
|zip_code|string|Zip code for the address of the building identified in the complaint.|
|med_inc_zip|integer|Median income imputed by zip code.|
|date_entered|date|Date complaint was filed|
|inspection_date|date|Date complaint was inspected|
|days_until_inspection|integer|Number of days between when complaint was filed and when it was officially inspected|


## Conclusions
We attempted a number of models, including a Generalized Linear Model (GLM), ARIMA models, and SARIMAX models. Despite our best efforts, there seems to be a significant disconnect between our features and our target, the number of days until a complaint was inspected.

Our best performing model was the __Random Forest__ with a training R^2 of 35.65% and a testing R^2 of a mere 7.32%. Though this model performed better than the other attempted models, it is still severely underfit and has a high variance. Therefore, __we would not advocate using this model as a practical solution to our problem.__

While there could certainly be improvements in our workflow, we feel the most likely answer to our original problem is __predicting the number of days until a complaint will be inspected is not closely linked to the features from the Departement of Buildings Complaint dataset or the median income of an area__. 

## Recommendations
It should be noted during this project we performed our analysis and modeling on a fraction of the entire dataset. This was due to practical computational limitations. We did our best to get a randomized sample from all observations within the dataset, though it is possible our subset of data was biased and lead to poor results. Therefore, one of the most important next steps to take should be an __adaptation to cloud-based computing__ that can better handle the size of this dataset. 

We also assumed inclusion of median income data would be beneficial, however, __further outside research around demographics__ may be useful and can be incorporated into further iterations of this project. We should also be mindful of the ethical implications of using such features in our model.

Regarding the clustering we conducted, __further steps can be taken to attempt to optimize the clustering analysis.__ Given our process was limited in computing power, the move to cloud computing would also be beneficial in this regard; the K-prototypes algorithm would be ideal to use as a way of incorporating continuous data, such as income, and categorical data, which is the form most of our data is already in.

We might also be able to __improve time-series model performance by omitting or at least reducing the clearly significant impact on the tail end of our dataset.__

Lastly, we recognize there are a number of __improvements to be made to our time-series models,__ such as included features likely to have an impact on our target, such as weather, the total number of unresolved complaints as any given time interval, or economic data.

## Resources
- [Data Source](https://data.cityofnewyork.us/Housing-Development/DOB-Complaints-Received/eabe-havv)
- [Complaint Codes](https://www1.nyc.gov/assets/buildings/pdf/complaint_category.pdf)
- [Disposition Codes](https://www1.nyc.gov/assets/buildings/pdf/bis_complaint_disposition_codes.pdf)
- [Data Explains](https://docs.google.com/spreadsheets/d/10p0HLqinKbUrSjKaZC2E0ZTHDXgULT0K/edit#gid=1015257717)
