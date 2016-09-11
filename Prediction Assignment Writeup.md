---
title: "Prediction Assignment Writeup"
author: "Xiukun Zhao"
date: "September 11, 2016"
output: html_document
---



## Introduction 

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. The goal of this project is to use data from accelerometers on the belt, forearm, arm, and dumbell of six participants. They were asked to perform barbell lifts correctly and incorrectly in five different ways. The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har.

## Pre-processing Data

First, we read the training dataset and the testing dataset into R and identify "NA", "" and "#DIV/0!" as NA strings.

### Load data


```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
trainingData <- read.csv("pml-training.csv", header=T, na.strings=c("NA", "", "#DIV/0!"))
testingData <- read.csv("pml-testing.csv", header=T, na.string=c("NA", "", "#DIV/0!"))
```

### Clean data

Next, we remove all columns that contains NA and remove features that are not in the testing dataset. We also remove the first seven features because they are related to the time-series or are not numeric.


```r
features <- names(testingData[,colSums(is.na(testingData)) == 0])[8:59]
trainingData <- trainingData[,c(features,"classe")]
testingData <- testingData[,c(features,"problem_id")]
```

### Data partitioning
The training data is partitioned into a training data set (60% of the total cases) and a testing data set (40% of the total cases). 


```r
set.seed(32323)
inTrain <- createDataPartition(y=trainingData$classe, p=0.6, list=FALSE)
training <- trainingData[inTrain,]
testing <- trainingData[-inTrain,]
```

## Random Forest Model

### Build a model

We Use a random forest model to predict the classe variable. 


```r
set.seed(32345)
modelFit <- randomForest(classe ~ ., data = training, ntree = 1000)
```

### Get error estimates

The out of sample error should be small. The error is estimated using the testing sample. The following results show that the model has an out of sample accuracy of 0.99.


```r
classeCol <- grep("classe",names(testing))
predTest <- predict(modelFit, newdata = testing[,-classeCol], type="class")
confusionMatrix(predTest,testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2229   14    0    0    0
##          B    3 1503   16    0    0
##          C    0    1 1351   14    4
##          D    0    0    0 1272    5
##          E    0    0    1    0 1433
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9926          
##                  95% CI : (0.9905, 0.9944)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9906          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9987   0.9901   0.9876   0.9891   0.9938
## Specificity            0.9975   0.9970   0.9971   0.9992   0.9998
## Pos Pred Value         0.9938   0.9875   0.9861   0.9961   0.9993
## Neg Pred Value         0.9995   0.9976   0.9974   0.9979   0.9986
## Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
## Detection Rate         0.2841   0.1916   0.1722   0.1621   0.1826
## Detection Prevalence   0.2859   0.1940   0.1746   0.1628   0.1828
## Balanced Accuracy      0.9981   0.9936   0.9923   0.9942   0.9968
```

## Prediction

Finally, we predict the testing data as follows.


```r
prediction <- predict(modelFit, testingData, type = "class")
prediction
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```

## Conclusion

From the confusion matrix, the Random Forest model is very accurate. Because of that we could expect nearly all of the submitted test cases to be correct. 

