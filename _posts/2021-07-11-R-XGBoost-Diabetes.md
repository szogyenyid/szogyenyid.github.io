---
layout: post
title: "Dataset Analysis: Early stage diabetes risk prediction using XGBoost"
date: 2021-07-11 11:56:00 +0200
categories: # Learning | Data Science | Security | Meta | Stories
    - Data Science
tags:
    - r
    - xgboost
    - classification
    - medical
description: Predicting early stage diabetes using XGBoost (logistic gbtrees). The used dataset is from UCI Machine Learning Repository.
# last_modified_at: 2021-07-07 09:40:00 +0200
author: Daniel Szogyenyi
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
## 1:   27          0.0234301        0.00137925         0.1124822       0.08477984
{% endhighlight %}

### Create the model


{% highlight r %}
xgb1 <- xgb.train(params = params, data = trainData, nrounds = bestIter$iter, watchlist = list(test=testData,train=trainData), print_every_n = 10)
{% endhighlight %}



{% highlight text %}
## [1]	test-logloss:0.410762	train-logloss:0.382777 
## [11]	test-logloss:0.117395	train-logloss:0.047575 
## [21]	test-logloss:0.105336	train-logloss:0.024972 
## [27]	test-logloss:0.107779	train-logloss:0.021369
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
##          0 40  3
##          1  0 61
##                                         
##                Accuracy : 0.9712        
##                  95% CI : (0.918, 0.994)
##     No Information Rate : 0.6154        
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.9399        
##                                         
##  Mcnemar's Test P-Value : 0.2482        
##                                         
##             Sensitivity : 1.0000        
##             Specificity : 0.9531        
##          Pos Pred Value : 0.9302        
##          Neg Pred Value : 1.0000        
##              Prevalence : 0.3846        
##          Detection Rate : 0.3846        
##    Detection Prevalence : 0.4135        
##       Balanced Accuracy : 0.9766        
##                                         
##        'Positive' Class : 0             
## 
{% endhighlight %}
The model reached an accuracy of ~99%, with a high enough sensitivity and specifity.  
1 out of 40 people without diabetes was marked "having diabetes" and  
0 out of 64 people with diabetes was marked "not having diabetes".  

It's excellent that the latter number is 0 (so specificity is 100%), as it's better to mark healthy people as having diabetes, because further medical examinations can confirm or reject this prediction.

### Analysing the importance of features


{% highlight r %}
mat <- xgb.importance(feature_names = colnames(as.matrix(df[idx,1:16])), model = xgb1)
xgb.ggplot.importance(importance_matrix = mat, n_clusters = c(4,4))
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/assets/Rfig/unnamed-chunk-11-1.svg)

The most important factor in the determination of someone having diabetes is polyuria. Not surprising that polydipsia is high too (because of the strong correlation between the two), as if someone consumes a lot of water, they will have a lot of urine.
The second more important factors are gender and age.  
Alopecia should be considered as an important factor, and everything in the "Cluster 1" (marked with red) is close to irrelevant.
