---
layout: post
title:  "Default Prediction: Data Preprocessing"
date:   2020-03-12 
categories: jekyll update
---

1. Load Data and do some Research
```r
#load data
retail=read.csv("data3.csv", header=T,encoding='UTF-8')
```
Data Exploration
```r
#daoqi includes data with "dqrq" > 2020.2.14
daoqi=retail[retail$dq==1,]
head(daoqi)
#weiyue includes overdue data after daoqi
#anshi includes those who pay on time
weiyue=daoqi[daoqi$yq==1,]
anshi=daoqi[daoqi$yq==0,]
```
There are 77 columns in each set.
```r
dim(daoqi)#2345+1104nan
dim(anshi)#2289
dim(weiyue)#1160
``` 
anshi set has 2289 rows and weiyue set has 1160 rows. daoqi set has 2345 meaningful rows and 1104 blank rows.

2. Data Cleaning
Use weiyue[is.na(weiyue$khbh)==0,] to check NAN rows.
Delete columns with too many missing values:
```r
xx = c("txdq","txhyx","nh","jtyb","gzdwyb","hyzk","khzt","poxm","pogj","khdj","jzzk","mz","lltzpl","censored")
w = w[,!names(w) %in% xx]
```
3. Missing Values Visualization
(1)Use function md.pattern() in mice package to generate a table that shows the missing value pattern as a matrix or data frame.
```r
library(mice)
md.pattern(w)
```
<img src="D:\Users\Documents\GitHub\maggiechy.github.io\images\DefaultPredict-miceplot.png">

(2)Use function aggr() in VIM package
<img src="D:\Users\Documents\GitHub\maggiechy.github.io\images\DefaultPredict-aggrplot.png">

(3)Use function matrixplot() in VIM package
Light color indicates small value, dark color indicates large value, missing value is red. 
<img src="D:\Users\Documents\GitHub\maggiechy.github.io\images\DefaultPredict-matrixplot.png">

(4)Use function marginplot() in VIM package
Marginplot generates a scatter chart to show the missing value information of two variables at the boundary of the graph.
```r
marginplot(w[c("gyzk","nl")],pch=c(20),col=c("darkgray","red","blue"))
```
<img src="D:\Users\Documents\GitHub\maggiechy.github.io\images\DefaultPredict-marginplot.png">

```r
#delete blank rows
wy<-na.omit(w)
```
```r
#delete same rows in anshi set
#we want anshi and weiyue set to maintain the same dimension
xx = c("txdq","txhyx","nh","jtyb","gzdwyb","hyzk","khzt","poxm","pogj","khdj","jzzk","mz","lltzpl","censored")
a = anshi[,!names(anshi) %in% xx]
a=a[is.na(a$khbh)==0,]
```

Delete columns with to many NAN / unrelevant variables (eg. family address)
```r
xx = c("khbh","cpfl","txhyd","llfdfx","jg","khjg","bz","khxz","csrq","xw","jtdz","jtdh","hjdz","yddhhm","gzdwdz","gzdw","zcjb","hylx","zw","gj","lxyy","khjg.1","ssjg","djrq","pogzdz","yqbjye","scyqrq","qxrq","gzdwdh","zczqh","dq","glr","jnwkh")
wy = wy[,!names(wy) %in% xx]
#write.csv(wy,file = "D:/Users/Documents/RStudio/default-proj/weiyue.csv")
```
Also delete simultaneously these columns in anshi set.
