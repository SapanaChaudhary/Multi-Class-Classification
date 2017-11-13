### Multiclass classification on an unknown data set 

With Sneha Konnur (MS(Research), IIT Madras), Ravina Gelda (MediaTek, Taiwan)

This project was done as part of the course on Introduction to Machine Learning offered by Dr. Balaraman Ravindran in the fall of 2015. 

##### Objective
Training data corresponding to 100 classes was provided. The task was to perform multi-class classification resulting in the best mean F1-measure for the 100 classes. 

##### Exploratory Data Analysis
We did an initial exploration of the data to understand it better. We performed the following experiments in this regard:

- Obtained the class counts
The 68th class occurs a good 1000 times in the data. The other classes occur on an average around 133 times. This is a clear case of data imbalance. 
      
- Sparsity
The data at hand is sparse. The attributes have zeros in the range 7000-12000. Few of the attributes clearly look like noise with data points in as few as 6 out of 14332 rows.

- Variance in the data
A major portion of each of the attributes is around zero with very high values occuring very few times. Scaling makes the small values even more small.

- Eigen vector analysis
The eigen vectors don’t make much sense for the data, given its dimensions. Projection on a few selected eigen values is not conclusive. 

With these initial findings in mind, we went on to carry out feature selection for the data. 

##### Feature Selection
- We first performed feature selection in R. The cfs function from the Fselector library selected 71 features(referrred to as F1 hereafter) from the training data. Performance results on these selected features did not result in a good model. So, did feature extraction again, this time around with CARET package in R. 

- Using CARET we performed the following operations on the data
    - Near zero variance columns were deleted.
    - All the columns with correlation above 0.95 were deleted.
    - Checked for linear dependence between the columns: Non of the columns were linearly dependent.
    - Finally, we obtained 1968 features(hereafter referrred as F2)
    
- Used clustering in weka to visualize randomly slected 4-5 classes. Conclusion: Data looks higly noisy. 

##### Classification methods tried
- Linear classification with indicator matrix on the whole of the data set.
    - Result : Performs bad.
    - Conclusion : Use logistic regression or bring nonlinearity by using kernel methods.

- Decision trees with Matlab ctree command. Runtime was well over 20-25 minutes and the particular command does not produce good results.

- Applyied different support vector machine kernel methods from libSVM library.
    - Result : Linear kernel with worked the best in terms of accuracy of the model. The model buiding process however takes considerable amount of time.
    
- To deal with class imbalance problem, we built a separate classifier for 68th class and a multi class classifier for the rest of the classes. There was not much of a difference in the results. A little bit of reading gave the insight that SVMs can handle data imbalance quiet well.

- We wanted to speed up the model buiding process. So, used open source Liblinear library(from the developers of libSVM). On building the model with whole of training data using liblinear, the resulting accuracy was good. To check if removing a few noisy columns makes a difference, we removed 3 columns and the resulting model did improve on accuracy.

- Next we built model with the first set of selected features,F1 using L-1 regularized logistic regression. The accuracy of the resulting model was very low. In one of our codes we had wrongly taken first 71 columns of the the train matrix and scaled them with the maximum of F1. Wrong scaling of data seemed to work really well. I dont know if it was only for the training and the validation data Or it might have worked well for the test data alike, because we didnt submit the the output produced by this model. For L-1 regularized logistic regression used here, we had done different weighing of the cost parameter for the classes: 68th class was weighed around 5 times more than the the rest of the classes while buiding one vs all classification model. The clasifier gave accuracy around 72% with mean F-measure around 70 for our validation data. Howerver, we realized the normalization was not correct and didn’t go ahaead with buiding additional model and ensembling them. It was amazing to see how good the results obtained with the wrong normalization were.

- With F1’s debacle, we extracted the features again. Built L2-regularized L2-loss support vector classification (primal (referred to as A hereonwards), L2-regularized logistic regression (dual)(referred to as B hereonwards) and L2-regularized logistic regression(referred to as C hereonwards) models on F2 independently. Model C seemed to work best for the train data.

- To enhance accuracy we thought of implementing a boosted SVM : Built model C on F1 features. The model outputs probabilities for each of the classes. We thought of building additional models on highly uncorrelated sets of features and then add them to the first model,weighing their probabilies in such a way that the resulting probabailities are accurately representative of the final clases. However, we had issues modelling the loss function for this kind of model. Hence, couldn’t go ahead with the idea.

- Back to square one, we thought of doing random forest of A models now. We built 12 classifiers partining the feature space into 12 different intervals, with overlap between intervals. While carrying out this task, we observed constructing only model A for all the features was not as good as constructing model A+model B for the different sets. We took the mode of resulting models, ti get the final output. It didnt work well. We could however conclude different parts of the feature space give good classification results from different methods. So, a proper weighted ensemble of classifiers that adequately represent linearities and non linearities in the feature space can do the trick here.

- Finally, we tried bootstrapping logistic regressors.Divided the instances into 10 folds. Kept the 10th fold for validation. Took the mode of results and observed that the results were not good enough. We then randomly selected 10,000 instances and trained C classifier on them with parameters: ’-s 6 -c 18 -B 0.7 -w68 4.53 wi 0.97’. Cost(c-cost of constraint violation) and sensitivity(p-sensitivity of the loss function) parameter tuning consumed a lot of time. This model gave us our best F-measure(65.1%).

##### Conclusion
Time to get the best accuracy is never enough. We personally feel we could have tried these few things:
- We could use L1s sparsity to perform feature selection.
- We knew outliers were existent in the data. Removing them could improve the accuracy further.
- Features selected for building models in step 8 might have been representing the same part of the feature space. We can shuffle and then built different classifiers.
- We could do better modelling of loss function while using ensemble of classifiers.
