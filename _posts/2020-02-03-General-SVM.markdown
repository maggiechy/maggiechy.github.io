---
layout: post
title:  "General SVM Model"
date:   2020-02-03 
categories: jekyll update
---

The support vector machine (SVM) is a supervised learning method that generates input-output mapping functions from a set of labeled training data. 

The mapping function can be either a classification function, say, the category of the input data, or a regression function. For classification, nonlinear kernel functions are used to transform input data to a high-dimensional feature space in which the input data become more separable compared to the original input space. 

See more [information](http://cs229.stanford.edu/notes/cs229-notes3.pdf) about svm.

The following is about the general SVM used on iris dataset.

1. A simple svm model
```r
library(e1071)
data(iris)
```
```r
#x---feature variable, data except iris 5th column (Species)
x = iris[,-5]
#y---outcome variable, data in iris 5th column
y = iris[,5]
#build svm model
m = svm(x,y,kernel = "radial", gamma = if(is.vector(x)) 1 else 1/ncol(x))
summary(m)
```
When deciding the gamma coefficient, if feature vector is a vector,gamma=1, otherwise gamma=1/number of feature vectors.
This SVM model is C-classification. 
Gamma in the Gauss kernel function is 0.25.
The cost of constraint violation determined by this model is 1.
Three categories: setosa (8 support vectors), versicolor (22 support vectors) and virginica (21 support vectors).

```r
#conform the sample characteristic matrix used for prediction
x = iris[,1:4]
pred = predict(m,x)
table(pred,y)
```
```r
#Tune to find the best gamma and cost
st = tune(svm, train.x=x, train.y=y, kernel="radial", ranges=list(cost=10^(-1:2), gamma=c(.5,1,2)))
print(st)
newm <- svm(Species ~ ., data=iris, kernel="radial", cost=1, gamma=0.5)
summary(newm)
```
```r
pred <- predict(newm,x)
table(pred,y)
```
This model predicts all setosa flowers correctly; 48 of the versicolor flowers are correect while 2 of them are categorized into virginica; 48 of the virginica flowers are correect while 2 of them are categorized into versicolor. 


2. A more comprehensive svm model
```r
#x---feature variable, data except iris 5th column (Species)
x = iris[,-5]
#y---outcome variable, data in iris 5th column
y = iris[,5]
```
We initialize the prediction matrix's 3 dimensions as 150,3,4 and initialize the precision matrix's 2 dimensions as 3,4. We can change methods (C-classification, nu-classification) and kernels (linear, polynomial, radial, sigmoid).

C-classification + radial:
```r
#model1: weight for each category is the same (equal to the #1 model)
model1 = svm(x,y)
pred1 = predict(model1,x)
table(pred1,y)
```
Optimization:
```r
#model2: weight for versicolor and virginica * 200
weight = c(1,200,200)
names(weight) = c("setosa","versicolor","virginica")
model2 = svm(x,y,class.weights = weight)
pred2 = predict(model2,x)
table(pred2,y)
summary(model2)
```
```r
#model3: weight for versicolor and virginica * 500
weight = c(1,500,500)
names(weight) = c("setosa","versicolor","virginica")
model3 = svm(x,y,class.weights = weight)
pred3 = predict(model3,x)
table(pred3,y)
```
For model3, all predictions are correct.
Now we change a kernel.

C-classification + linear/polynomial/sigmoid:
```r
model4 = svm(Species~., data = iris, kernel = 'linear')
summary(model4)
plot(model4, 
     data = iris,
     Petal.Width ~ Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))
# predict
pred4 = predict(model4, iris)
summary(pred4)
#confusion matrix
table(pred4,y)
```
```r
model5 = svm(Species~., data = iris, kernel = 'polynomial')
summary(model5)
plot(model5, 
     data = iris,
     Petal.Width ~ Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))
# predict
pred5 = predict(model5, iris)
#summary(model5)
#confusion matrix
table(pred5,y)
```
```r
model6 = svm(Species~., data = iris, kernel = 'sigmoid')
summary(model6)
plot(model6, 
     data = iris,
     Petal.Width ~ Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))
# predict
pred6 = predict(model6, iris)
#summary(model6)
#confusion matrix
table(pred6,y)
```

An algorithmn for comparing the performance of different SVMs---Leave-one out experiment

There are n positive samples and x samples as background. We picks 1 of the n positive samples and mix it randomly into half of the x samples. We build a model from the n-1 sample set and use this model to predict which samples in the x+1 sample set contain the positive signal---exhibit the same features showed in the n-1 sample set. The accuracy is evaluated via AUC (area under the ROC curve).

We compare SVDD (a one-class model) and general SVM. Here we use the entire unlabeled background set as negative examples.

Conclusion: We note the general upward trend in the performance of one-class models as alpha increases. Because a larger mixing proportion of the positive sample makes the mixtures look more like the positive class. The trend is not shared by v-SVM models, which are unable to identify samples with mixed stemness signal from others in the background. A potential explanation for the poor performance comes from sample locality in high-dimensional feature spaces. "An SVM can be viewed as a mechanical system, where the decision plane is a "stiff sheet" in mechanical equilibrium, upon which the training samples exert forces and torques. This effectively makes the model highly sensitive to noise and reduces its generalization to the unsampled portions of the feature space."