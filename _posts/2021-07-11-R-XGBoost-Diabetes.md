---
layout: post
title: "Dataset Analysis: Early stage diabetes risk prediction using XGBoost"
date: 2021-07-11 11:56:00 +0200
category: data-science # Learning | Data Science | Security | Meta | Stories 
tags:
    - r
    - xgboost
    - classification
    - medical
description: Predicting early stage diabetes using XGBoost (logistic gbtrees). The used dataset is from UCI Machine Learning Repository.
last_modified_at: 2021-07-15 12:21:00 +0200
author: Daniel Szogyenyi
readtime: 6
---

## Introducing the dataset

The data I used is from the [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Early+stage+diabetes+risk+prediction+dataset.#). The set has been collected using direct questionnaires from the patients of Sylhet Diabetes Hospital in Sylhet, Bangladesh and approved by a doctor.

## Including libraries


{% highlight r %}
library(readr) # for reading csv
library(xgboost) # for the prediction model
library(caret) # for data separation and confusion matrix
{% endhighlight %}

## Reading the file


{% highlight r %}
df <- read_csv("src/diabetes.csv", col_types = cols())
head(df)
{% endhighlight %}



{% highlight text %}
## # A tibble: 6 x 17
##     Age Gender Polyuria Polydipsia `sudden weight l… weakness Polyphagia `Genital thrush` `visual blurrin… Itching Irritability `delayed healin… `partial paresi… `muscle stiffne…
##   <dbl> <chr>  <chr>    <chr>      <chr>             <chr>    <chr>      <chr>            <chr>            <chr>   <chr>        <chr>            <chr>            <chr>           
## 1    40 Male   No       Yes        No                Yes      No         No               No               Yes     No           Yes              No               Yes             
## 2    58 Male   No       No         No                Yes      No         No               Yes              No      No           No               Yes              No              
## 3    41 Male   Yes      No         No                Yes      Yes        No               No               Yes     No           Yes              No               Yes             
## 4    45 Male   No       No         Yes               Yes      Yes        Yes              No               Yes     No           Yes              No               No              
## 5    60 Male   Yes      Yes        Yes               Yes      Yes        No               Yes              Yes     Yes          Yes              Yes              Yes             
## 6    55 Male   Yes      Yes        No                Yes      Yes        No               Yes              Yes     No           Yes              No               Yes             
## # … with 3 more variables: Alopecia <chr>, Obesity <chr>, class <chr>
{% endhighlight %}

## Preprocessing

### Cast all types to logical

Gender is mutated into "isMale", "Female" leading to FALSE, and "Male" to TRUE.


{% highlight r %}
df[df == "Yes" | df=="Positive" | df=="Male"] <- "T"
df[df == "No" | df=="Negative" | df=="Female"] <- "F"

df[2:17] <- apply(df[2:17],2,function(c){
  as.logical(unlist(c))
})

df$isMale <- df$Gender
df$Diabetes <- df$class
df$class <- NULL
df$Gender <- NULL
{% endhighlight %}

### Separate the dataset

Will use 80% of the data for training, and 20% for testing. "Diabetes" ratio in the subsets remain the same as in the whole set.


{% highlight r %}
idx <- caret::createDataPartition(df$Diabetes, p=0.8, list=F)
# Use every column but "diabetes" as data, and "diabetes" as label.
trainData <- xgb.DMatrix(data = as.matrix(df[idx,1:16]), label = unlist(df[idx,17]))
testData <- xgb.DMatrix(data = as.matrix(df[-idx,1:16]), label = unlist(df[-idx,17]))
{% endhighlight %}

## Creating the model

### Set up parameters


{% highlight r %}
params <- list(
  booster = "gbtree", # gradboosted trees
  objective = "binary:logistic", # as "having diabetes" is a binary attribute, use logistic
  eval_metric="logloss", # because the objective is logistic
  eta=0.5,
  gamma=0,
  max_depth=10,
  min_child_weight=1,
  subsample=1,
  colsample_bytree=1
)
{% endhighlight %}

### Cross-validate to get the ideal number of iterations


{% highlight r %}
xgbcv <- xgb.cv(
  params = params,
  data = trainData,
  nrounds = 500,
  nfold = 10,
  showsd = T,
  stratified = T,
  verbose=F
)
{% endhighlight %}

### Get the ideal number of iterations


{% highlight r %}
bestIter <- xgbcv$evaluation_log[xgbcv$evaluation_log$test_logloss_mean == min(xgbcv$evaluation_log$test_logloss_mean)]
print(bestIter)
{% endhighlight %}



{% highlight text %}
##    iter train_logloss_mean train_logloss_std test_logloss_mean test_logloss_std
## 1:   87          0.0133054      0.0004704235         0.1035381       0.09787715
{% endhighlight %}

### Create the model


{% highlight r %}
xgb1 <- xgb.train(params = params, data = trainData, nrounds = bestIter$iter, watchlist = list(test=testData,train=trainData), print_every_n = 10)
{% endhighlight %}



{% highlight text %}
## [1]	test-logloss:0.374550	train-logloss:0.385840 
## [11]	test-logloss:0.072249	train-logloss:0.048993 
## [21]	test-logloss:0.052321	train-logloss:0.027817 
## [31]	test-logloss:0.044564	train-logloss:0.021101 
## [41]	test-logloss:0.042983	train-logloss:0.017620 
## [51]	test-logloss:0.038606	train-logloss:0.015633 
## [61]	test-logloss:0.035977	train-logloss:0.014309 
## [71]	test-logloss:0.033654	train-logloss:0.013530 
## [81]	test-logloss:0.034843	train-logloss:0.012535 
## [87]	test-logloss:0.033825	train-logloss:0.012278
{% endhighlight %}

## Evaluation of the model

### Prediction on test dataset

Let's predict the test dataset.  
As logistic regression returns the probability of being true, if the value is larger than 0.5, I should say that the person has diabetes.


{% highlight r %}
xgbpred <- predict(xgb1,testData)
xgbpred <- as.numeric(xgbpred>0.5)
{% endhighlight %}

### Analysing the confusion matrix


{% highlight r %}
confusionMatrix(as.factor(xgbpred), as.factor(as.numeric(unlist(df[-idx,17]))))
{% endhighlight %}



{% highlight text %}
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction  0  1
##          0 40  2
##          1  0 62
##                                           
##                Accuracy : 0.9808          
##                  95% CI : (0.9323, 0.9977)
##     No Information Rate : 0.6154          
##     P-Value [Acc > NIR] : <2e-16          
##                                           
##                   Kappa : 0.9598          
##                                           
##  Mcnemar's Test P-Value : 0.4795          
##                                           
##             Sensitivity : 1.0000          
##             Specificity : 0.9688          
##          Pos Pred Value : 0.9524          
##          Neg Pred Value : 1.0000          
##              Prevalence : 0.3846          
##          Detection Rate : 0.3846          
##    Detection Prevalence : 0.4038          
##       Balanced Accuracy : 0.9844          
##                                           
##        'Positive' Class : 0               
## 
{% endhighlight %}
The model reached an accuracy of ~98%, with a high enough sensitivity and specifity.  
0 out of 40 people without diabetes was marked "having diabetes" and  
2 out of 64 people with diabetes was marked "not having diabetes".  

It's excellent that the latter number is so low (specificity is 97%), as it's better to mark healthy people as having diabetes, because further medical examinations can confirm or reject this prediction. 100% specifity would be ideal, even with the decrease of sensitivity, so when fine-tuning the model, this is the thing to keep in mind.

### Analysing the importance of features


{% highlight r %}
mat <- xgb.importance(feature_names = colnames(as.matrix(df[idx,1:16])), model = xgb1)
xgb.ggplot.importance(importance_matrix = mat, n_clusters = c(4,4))
{% endhighlight %}

![plot of chunk xgboost-diabetes-params](/assets/Rfig/xgboost-diabetes-params-1.svg)

The most important factor in the determination of someone having diabetes is polydipsia. Not surprising that polyuria is high too, because of the strong correlation between the two (as if someone consumes a lot of water, they will have a lot of urine).
The second more important factors are gender and age.  
Delayed healing and alopecia may be considered as somehow-important factors, but everything in the "Cluster 1" (marked with red) is close to irrelevant.
