---
layout: post
title:  "One Class SVM Model"
date:   2020-02-02 23:52:38
categories: jekyll update
---

One class problem, is not aimed to distinguish between different categories, but to generate a description of a certain category of data thus creating a region in the sample space. If our sample falls outside this region, then the sample does not belong to this category.

When we are faceed with a classification problem, and there is only one type of samples or there are two types of samples (but the number of one type sample of far smaller than another) (if a binary classifier is exploited, the positive and negative samples in the training set will cause the classifier to be too biased towards a large number of sample category), we can consider using one-class classification.

For example, we extract the characteristics of human's normal reactions in the bank and use a one-type svm to express them. For an abnormal behaviour, we can quickly conclude that its features are inconsistent with what we have extracted from normal behaviours, so it is definitely not a normal reaction in the bank. Then an alarm is issued. 

The following is the use of one-class SVM on iris dataset.

1. A general ocsvm model.
```r
library(e1071)
data(iris)
data<-iris
```
```r
data<-subset(data,Species=='versicolor')
x<-subset(data,select=-Species)#make x variables
y<-data$Species#make y variable
model<-svm(x,y,type='one-classification')
print(model)
summary(model)
```
```r
#test on the whole set
predict(model,subset(iris,select=-Species))
```

2. A more elaborate model.
```r
library(e1071)
library(caret)
library(NLP)
library(tm)
data(iris)
```
```r
iris$SpeciesClass[iris$Species=="versicolor"] <- "TRUE"
iris$SpeciesClass[iris$Species!="versicolor"] <- "FALSE"
print(iris$SpeciesClass)
```
```r
trainPositive<-subset(iris,SpeciesClass=="TRUE")
testNegative<-subset(iris,SpeciesClass=="FALSE")

# 60% of the "TRUE" dataset is made training set
inTrain<-createDataPartition(1:nrow(trainPositive),p=0.6,list=FALSE)

trainpredictors<-trainPositive[inTrain,1:4]
trainLabels<-trainPositive[inTrain,6]
```
```r
# 40% of the "TRUE" data and all "FALSE" data are made test set
testPositive<-trainPositive[-inTrain,]
test<-rbind(testPositive,testNegative)

testpredictors<-test[,1:4]
testLabels<-test[,6]
```
```r
#one-class svm model
ocsvm.model<-svm(trainpredictors,y=NULL,
               type='one-classification',
               nu=0.10,
               scale=TRUE,
               kernel="radial")

#predict the trainging dataset and the test dataset
ocsvm.predtrain<-predict(ocsvm.model,trainpredictors)
ocsvm.predtest<-predict(ocsvm.model,testpredictors)

confTrain<-table(Predicted=ocsvm.predtrain,Reference=trainLabels)
confTest<-table(Predicted=ocsvm.predtest,Reference=testLabels)

confusionMatrix(confTest,positive='TRUE')
```
```r
#apply the model to the test set
print(confTrain)
print(confTest)
```
See the test dataset prediction result, the accuracy is 94.07% and there are totally 3+4=7 wrong-predicted data. 