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
df <- read_csv("diabetes.csv",col_types = cols())
{% endhighlight %}



{% highlight text %}
## Error: 'diabetes.csv' does not exist in current working directory ('/home/soegaeni/blog/szogyenyid.github.io/_Rmd').
{% endhighlight %}



{% highlight r %}
head(df)
{% endhighlight %}



{% highlight text %}
##                                               
## 1 function (x, df1, df2, ncp, log = FALSE)    
## 2 {                                           
## 3     if (missing(ncp))                       
## 4         .Call(C_df, x, df1, df2, log)       
## 5     else .Call(C_dnf, x, df1, df2, ncp, log)
## 6 }
{% endhighlight %}

# Preprocessing

## Cast all types to logical

Gender is mutated into "isMale", "Female" leading to FALSE, and "Male" to TRUE.


{% highlight r %}
df[df == "Yes" | df=="Positive" | df=="Male"] <- "T"
{% endhighlight %}



{% highlight text %}
## Error in df == "Yes": comparison (1) is possible only for atomic and list types
{% endhighlight %}



{% highlight r %}
df[df == "No" | df=="Negative" | df=="Female"] <- "F"
{% endhighlight %}



{% highlight text %}
## Error in df == "No": comparison (1) is possible only for atomic and list types
{% endhighlight %}



{% highlight r %}
df[2:17] <- apply(df[2:17],2,function(c){
  as.logical(unlist(c))
})
{% endhighlight %}



{% highlight text %}
## Error in df[2:17]: object of type 'closure' is not subsettable
{% endhighlight %}



{% highlight r %}
df$isMale <- df$Gender
{% endhighlight %}



{% highlight text %}
## Error in df$Gender: object of type 'closure' is not subsettable
{% endhighlight %}



{% highlight r %}
df$Diabetes <- df$class
{% endhighlight %}



{% highlight text %}
## Error in df$class: object of type 'closure' is not subsettable
{% endhighlight %}



{% highlight r %}
df$class <- NULL
{% endhighlight %}



{% highlight text %}
## Error in df$class <- NULL: object of type 'closure' is not subsettable
{% endhighlight %}



{% highlight r %}
df$Gender <- NULL
{% endhighlight %}



{% highlight text %}
## Error in df$Gender <- NULL: object of type 'closure' is not subsettable
{% endhighlight %}

## Separate the dataset

Will use 80% of the data for training, and 20% for testing. "Diabetes" ratio in the new sets remain the same as in the whole set.


{% highlight r %}
idx <- caret::createDataPartition(df$Diabetes, p=0.8, list=F)
{% endhighlight %}



{% highlight text %}
## Error in df$Diabetes: object of type 'closure' is not subsettable
{% endhighlight %}



{% highlight r %}
# Use every column but "diabetes" as data, and "diabetes" as label.
trainData <- xgb.DMatrix(data = as.matrix(df[idx,1:16]), label = unlist(df[idx,17]))
{% endhighlight %}



{% highlight text %}
## Error in as.matrix(df[idx, 1:16]): object 'idx' not found
{% endhighlight %}



{% highlight r %}
testData <- xgb.DMatrix(data = as.matrix(df[-idx,1:16]), label = unlist(df[-idx,17]))
{% endhighlight %}



{% highlight text %}
## Error in as.matrix(df[-idx, 1:16]): object 'idx' not found
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



{% highlight text %}
## Error in xgb.cv(params = params, data = trainData, nrounds = 500, nfold = 10, : object 'trainData' not found
{% endhighlight %}

## Let's see the ideal number of iterations


{% highlight r %}
bestIter <- xgbcv$evaluation_log[xgbcv$evaluation_log$test_logloss_mean == min(xgbcv$evaluation_log$test_logloss_mean)]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'xgbcv' not found
{% endhighlight %}



{% highlight r %}
print(bestIter)
{% endhighlight %}



{% highlight text %}
## Error in print(bestIter): object 'bestIter' not found
{% endhighlight %}

## Create the model


{% highlight r %}
xgb1 <- xgb.train (params = params, data = trainData, nrounds = bestIter$iter, watchlist = list(val=testData,train=trainData), print_every_n = 10)
{% endhighlight %}



{% highlight text %}
## Error in xgb.train(params = params, data = trainData, nrounds = bestIter$iter, : object 'trainData' not found
{% endhighlight %}

# Evaluation of the model

## Prediction on test dataset

Let's predict the test dataset.  
As logistic regression returns the probability of being true, if the value is larger than 0.5, let's say the person has diabetes.


{% highlight r %}
xgbpred <- predict (xgb1,testData)
{% endhighlight %}



{% highlight text %}
## Error in predict(xgb1, testData): object 'xgb1' not found
{% endhighlight %}



{% highlight r %}
xgbpred <- ifelse (xgbpred > 0.5,1,0)
{% endhighlight %}



{% highlight text %}
## Error in ifelse(xgbpred > 0.5, 1, 0): object 'xgbpred' not found
{% endhighlight %}

## Analysing the confusion matrix


{% highlight r %}
confusionMatrix (as.factor(xgbpred), as.factor(as.numeric(unlist(df[-idx,17]))))
{% endhighlight %}



{% highlight text %}
## Error in is.factor(x): object 'xgbpred' not found
{% endhighlight %}
The model reached an accuracy of ~96%, with a high enough sensitivity and specifity.  
2 out of 40 people without diabetes was marked "having diabetes" and  
0 out of 64 people with diabetes was marked "not having diabetes".  

It's excellent that the latter number is 0 (so specificity is 100%), as it's better to mark healthy people as ill, because further medical examinations can confirm or reject this prediction.

## Analysing the importance of features


{% highlight r %}
mat <- xgb.importance(feature_names = colnames(as.matrix(df[idx,1:16])), model = xgb1)
{% endhighlight %}



{% highlight text %}
## Error in xgb.importance(feature_names = colnames(as.matrix(df[idx, 1:16])), : object 'xgb1' not found
{% endhighlight %}



{% highlight r %}
xgb.ggplot.importance(importance_matrix = mat, n_clusters = c(4,4))
{% endhighlight %}



{% highlight text %}
## Error in is.data.table(importance_matrix): object 'mat' not found
{% endhighlight %}

The most important factor in the determination of someone having diabetes is polydipsia.  
The second more important factors are gender and age (and polyuria, but it has a strong correlation with polydipsia obviously).  
Alopecia, polyphagia and irritability should be considered as important factors, and everything in the "Cluster 1" (marked with red) is close to irrelevant.
