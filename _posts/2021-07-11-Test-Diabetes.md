---
title: "Predicting type 2 diabetes based on age, gender, and symptoms"
author: "Dániel Szőgyényi"
date: "2021.07.02."
output: html_notebook
---

# Including libraries


{% highlight r %}
library(readr) # for reading csv
library(xgboost) # for the prediction model
library(caret) # for data separation and confusion matrix
{% endhighlight %}

# Reading the file


{% highlight r %}
df <- read_csv("~/Documents/Kaggle_Datasets/diabetes.csv",col_types = cols())
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

# Preprocessing

## Cast all types to logical

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

## Separate the dataset

Will use 80% of the data for training, and 20% for testing. "Diabetes" ratio in the new sets remain the same as in the whole set.


{% highlight r %}
idx <- caret::createDataPartition(df$Diabetes, p=0.8, list=F)
# Use every column but "diabetes" as data, and "diabetes" as label.
trainData <- xgb.DMatrix(data = as.matrix(df[idx,1:16]), label = unlist(df[idx,17]))
testData <- xgb.DMatrix(data = as.matrix(df[-idx,1:16]), label = unlist(df[-idx,17]))
{% endhighlight %}

# Creating the model

## Set up parameters and a cross-validation


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

## Let's see the ideal number of iterations


{% highlight r %}
bestIter <- xgbcv$evaluation_log[xgbcv$evaluation_log$test_logloss_mean == min(xgbcv$evaluation_log$test_logloss_mean)]
print(bestIter)
{% endhighlight %}



{% highlight text %}
##    iter train_logloss_mean train_logloss_std test_logloss_mean test_logloss_std
## 1:  108          0.0126502      0.0005526738         0.1201801       0.07066102
{% endhighlight %}

## Create the model


{% highlight r %}
xgb1 <- xgb.train (params = params, data = trainData, nrounds = bestIter$iter, watchlist = list(val=testData,train=trainData), print_every_n = 10)
{% endhighlight %}



{% highlight text %}
## [1]	val-logloss:0.397396	train-logloss:0.377595 
## [11]	val-logloss:0.070882	train-logloss:0.050304 
## [21]	val-logloss:0.045248	train-logloss:0.028858 
## [31]	val-logloss:0.041388	train-logloss:0.021677 
## [41]	val-logloss:0.039579	train-logloss:0.017741 
## [51]	val-logloss:0.038201	train-logloss:0.015684 
## [61]	val-logloss:0.037961	train-logloss:0.014254 
## [71]	val-logloss:0.038008	train-logloss:0.013387 
## [81]	val-logloss:0.038218	train-logloss:0.012884 
## [91]	val-logloss:0.037395	train-logloss:0.012472 
## [101]	val-logloss:0.037695	train-logloss:0.012215 
## [108]	val-logloss:0.037151	train-logloss:0.011825
{% endhighlight %}

# Evaluation of the model

## Prediction on test dataset

Let's predict the test dataset.  
As logistic regression returns the probability of being true, if the value is larger than 0.5, let's say the person has diabetes.


{% highlight r %}
xgbpred <- predict (xgb1,testData)
xgbpred <- as.numeric(xgbpred>0.5)
{% endhighlight %}

## Analysing the confusion matrix


{% highlight r %}
confusionMatrix (as.factor(xgbpred), as.factor(as.numeric(unlist(df[-idx,17]))))
{% endhighlight %}



{% highlight text %}
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction  0  1
##          0 40  1
##          1  0 63
##                                           
##                Accuracy : 0.9904          
##                  95% CI : (0.9476, 0.9998)
##     No Information Rate : 0.6154          
##     P-Value [Acc > NIR] : <2e-16          
##                                           
##                   Kappa : 0.9798          
##                                           
##  Mcnemar's Test P-Value : 1               
##                                           
##             Sensitivity : 1.0000          
##             Specificity : 0.9844          
##          Pos Pred Value : 0.9756          
##          Neg Pred Value : 1.0000          
##              Prevalence : 0.3846          
##          Detection Rate : 0.3846          
##    Detection Prevalence : 0.3942          
##       Balanced Accuracy : 0.9922          
##                                           
##        'Positive' Class : 0               
## 
{% endhighlight %}
The model reached an accuracy of ~99%, with a high enough sensitivity and specifity.  
1 out of 40 people without diabetes was marked "having diabetes" and  
0 out of 64 people with diabetes was marked "not having diabetes".  

It's excellent that the latter number is 0 (so specificity is 100%), as it's better to mark healthy people as ill, because further medical examinations can confirm or reject this prediction.

## Analysing the importance of features


{% highlight r %}
mat <- xgb.importance(feature_names = colnames(as.matrix(df[idx,1:16])), model = xgb1)
xgb.ggplot.importance(importance_matrix = mat, n_clusters = c(4,4))
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/assets/Rfig/unnamed-chunk-10-1.svg)

The most important factor in the determination of someone having diabetes is polyuria. Not surprising that polydipsia is high too (because of the strong correlation betwen the two), as if someone consumes a lot of water, they will have a lot of urine.
The second more important factors are gender and age.  
Alopecia should be considered as an important factor, and everything in the "Cluster 1" (marked with red) is close to irrelevant.
