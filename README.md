# Breast-Cancer-Data-Set-Analysis
Code to construct a benign/non-benign classifier for breast cancer tumors. Data pulled from a dataset that was built upon the Wisconsin breast cancer data set (https://archive.ics.uci.edu/ml/datasets/breast+cancer+wisconsin+(original))

This notebook is a cookbook on how to get a class-imbalanced classifier up and running 


## Step 1: Understand the data

That dataset we are working with describes a set of breast tumor data that was collected by a group of researchers at University of Wisconsin Hospitals, who analyzed samples from a series of patients over some years.

The features of each tumor were placed on a 1-10 scale that was determined by researchers and applied consistently across the patients.
We want to predict whether a tumor sample is likely to be benign or malicious so health care professionals can get patients the care they need as quickly as possible without delays caused with biopsies, etc. Also when determining how to que samples for biopsies, likelihood of being malicious may be one component to consider.

We would like the classifier to have high recall since there is a significant cost of failing to correctly measure a malicious tumor as malicious

## Step 2: Perform EDA

Some data rows have missing or improperly formatted data. For example, the most common error is adding a 0 to the end of feature values, i.e. '20' instead of '2'.

This mistake is easily corrected by post-processing. There are missing data beyond this, which is more difficult to correct for. Missing data occurs in just 100 rows of our 16,000 row database, so we will enjoy this for now, although processes like KNN imputation or mean imputation may be considered later on if needed.

Analyzing a training set of the data, we can observe the distribution of our data and how that distribution changes between classes. Right off the bat, tumor size and uniformity seems to be highly indicative of tumor class.


![alt text](https://github.com/prateekpuri01/Breast-Cancer-Data-Set-Analysis/blob/master/plots/cell_size_hist.png)

We also want to remove any multicolinearity of features before constructing our model. We can look at a R^2 matrix for our features and also look the VIF factors for our features. After analyzing the data, both size and shape uniformity were high correlated (VIF>15), so I removed shape uniformity from the feauture set. After doing this, all VIF's for features are below 10. I removed shape uniformity instead of size uniformity because size uniformity had a greater correaltion with class outcome and thus seemed more important for classification

## Step 3: Prepare data for modeling

The data have difficult scales and therefore I want to standardize them before inputting in my model so that regularization is treated equally across features. I chose a typical standard scaler for this purpose and analyzed the feature distributions after scaling to make sure they are reasonable. The Mitoses feature has a slightly exponential distribution and one could consider a log transform here but as we will see, this feature is not dominant in our model and thus the transform was not considered here

There is also a significant 100: 1 benign:malicious class imbalance here, so I want to upsample my training set's minority class to account for this. I did this by resampling from my minority class until a 1:1 ratio was established

## Step 4: Picking and tuning a model

I elected to go with a standard logisitc regression model here for ease of implementation. RFC could also be chosen if more complex feature interactions are expected. We do not have too many features so one could argue for not using regularization to limit bias, but I will leave this parameter as a hyper-parameter and perform a grid search on my train/dev set to see which value to use.

For this grid search, I used recall as a metric and came up with a value of C value of .1

## Step 5: Analyze results

One metric to consider is recall. 

On both my validation and test set, I saw a recall of .97 for my malicious tumor class, while maintained a precision of .9 for my malicious class on my test set. The fact that there appears to be little variance between validation/test sets is encouraging. Looking at the ROC, I see a AUC of .98 on my test set, which is also encouraging 

We can also look at how important each feature is. For this, I elected to use sklearn's permutation feature importance feature. Which essentially scrambles the data for a particular model and then computes how a metric of choice, i.e. recall, changes for the scrambled data. The idea is if your model performance didn't change much when a feature was scrambled (i.e. values for this feature were randomly chosen from feature set), the feature must not have been very important. 

Here is plot displaying the relative importance of each feature according to this method, which turns out to offer a slightly different result than if you were just looking at the coefficients of the LR model (perhaps due to a small degree of multi-colinearity existing in the data)


As we suspected from our EDA, tumor size and uniformity seem to dominate the classifier

## Step 6: Failure cases

We can look at the feature distribution for our false positive/false negatives/true positives/true negatives to get a sense of where our model failed and where it succeeded. As seen from the violin plots, the true positives seems to have the common theme of having small tumor sizes and low tumor uniformity, while the true negatives are the opposite. 

The false-positive and false-negative cases seem to buck these trends to some degree. 
The cases we are most concerned with - false negatives - seems to have low clump size but high uniformity, something that confused the classifier. It's possible that a non-linear model, like a RFC, could pick up on this type of interaction more easily. But we also only had 3 false negatives in our training set total out of ~120 total positive cases, so the statistics are small. Due to the ease of interpretability, I'm going to stick with the linear model for now although further performance gains might be possible


