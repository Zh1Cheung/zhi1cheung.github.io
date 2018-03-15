---
title: Machine Learning
categories:
- ML
---



它是人工智能的核心，是使计算机具有智能的根本途径，其应用遍及人工智能的各个领域，它主要使用归纳、综合而不是演绎。


## 数学是基础
微积分
概率论和统计学
线性代数（矩阵，向量）
数值数学（数值分析，线性规划，凸优化理论，常见数值优化算法）
实分析和泛函的基础

《统计学习方法》 李航博士



## 机器学习

机器学习 机器学习是近20多年兴起的一门多领域交叉学科，涉及概率论、统计学、逼近论、凸分析、算法复杂度理论等多门学科。机器学习理论主要是设计和分析一些让计算机可以自动“学习”的算法。机器学习算法是一类从数据中自动分析获得规律，并利用规律对未知数据进行预测的算法。因为学习算法中涉及了大量的统计学理论，机器学习与统计推断学联系尤为密切，也被称为统计学习理论。算法设计方面，机器学习理论关注可以实现的，行之有效的学习算法。

从大量数据中找到规律和知识，然后用规律和知识做预测和决策。


### 监督学习

从给定的训练数据集中学习一个函数，当新数据到来时，可以根据这个函数预测结果。

Square判断一笔交易是否是欺诈，信用卡咋骗
Airbnb 预测用户网络平台上的违规操作
Airbnb 搜索引擎，搜索结果最大程度可能是用户感兴趣的租房
Airbnb 预测一个房租的最佳定价范围


线性回归
决策树


Aerosolve
Scikit-Learn python


### 无监督学习

从一个输入集里选择识别隐藏的有用的信息，比如从生物信息的DNA里找到负责同一个生物功能的DNA群，图像图形处理里的人脸识别。研究方法是聚类分析。

*	数据处理与可视化：PCA，LDA，MDS
*	聚类算法
*	稀疏编码


### 增强学习

通过观察来学习应该如何的动作。



机器学习偏向数学问题推导，数据挖掘就是抽特征
不要沉迷于数学公式推导，理解如何运用数据
动手实现一些简单的算法，如感知机，k近邻，线性回归
找一个实际案例，从他的算法选择，特征参数选取调整，以及数据管道的建立等系统的学习一下


基本不会去实现这些基础算法，都有现成的开源工具

概率图模型（Probabilistic graphical model）
两个核心的机器学习模型：Latent Dirichlet Allocation（LDA） Probabilistic Matrix Factorization（PMF）
统计计算（Statistical computing）	
深度学习（Deep learning）
优化（optimization）
PAC学习理论（PAC Learning）
非参数贝叶斯统计（Non-parametric Bayesian statistics）

参考链接：http://www.zhihu.com/question/21714701





分类器

*	Naive Bayes
*	Linear Discriminant Analysis
*	Logistic Regression
*	Linear SVM
*	Kernel SVM
*	Adaboost
*	Decision
*	Neural network









### 数据挖掘

在知乎的描述中。`数据挖掘`是指从大量的数据中自动搜索隐藏于其中的有着特殊关系性的信息和知识的过程。



## 入门参考


[Machine learning](https://www.coursera.org/learn/machine-learning/) Andrew Ng 在 coursera公开课。最好能完成所有作业。课程讲义http://cs229.stanford.edu/materials.html

网易的andrew ng公开课， 这个也不错，年代久远一些。

[julyedu 课程大纲](http://julyedu.com/course/getDetail?course_id=34#discard)

《机器学习基石》 https://www.coursera.org/course/ntumlone   

《机器学习技法》 公开课

《机器学习实战》

《机器学习：实用案例解析》 本书比较全面系统地介绍了机器学习的方法和技术。全书案例既有分类问题，也有回归问题；既包含监督学习，也涵盖无监督学习。本书讨论的案例从分类讲到回归，然后讨论了聚类、降维、最优化问题等。这些案例包括分类：垃圾邮件识别，排序：智能收件箱，回归模型：预测网页访问量，正则化：文本回归，最优化：密码破解，无监督学习：构建股票市场指数，空间相似度：用投票记录对美国参议员聚类，推荐系统：给用户推荐R语言包，社交网络分析：在Twitter上感兴趣的人，模型比较：给你的问题找到最佳算法。各章对原理的叙述力求概念清晰、表达准确，突出理论联系实际，富有启发性，易于理解。在探索这些案例的过程中用到的基本工具就是R统计编程语言。


《Machine Learning》 Tom Mitchell 
Simon Haykin的《神经网络与机器学习》




---
第1课 微积分与概率论
Taylor展式、梯度下降和牛顿法初步、Jensen不等式、常见分布与共轭分布

第2课 数理统计与参数估计
切比雪夫不等式、大数定理、中心极限定理、矩估计、极大似然估计

第3课 矩阵和线性代数
特征向量、对称矩阵对角化、线性方程

第4课 凸优化
凸集、凸函数、凸优化、KKT条件

第5课 回归
最小二乘法、高斯分布、梯度下降、过拟合、Logistic回归
实践示例：线性回归、Logistic回归实现和分析

第6课 梯度下降算法剖析
自适应学习率、拟牛顿、LBFGS
实践示例：自适应学习率代码实现和参数调试分析

第7课 最大熵模型
熵、相对熵、信息增益、最大熵模型、IIS

第8课 聚类
K-means/K-Medoid/密度聚类/谱聚类
实践示例：K-means、谱聚类代码实现和参数调试分析

第9课 推荐系统
协同过滤、隐语义模型pLSA/SVD、随机游走Random Walk
实践示例：协同过滤代码实现和参数调试分析

第10课 决策树和随机森林
ID3、C4.5、CART、Bagging、GBDT
实践案例：使用随机森林进行数据分类

第11课 Adaboost
Adaboost、前向分步算法

第12课 SVM
线性可分支持向量机、线性支持向量机、非线性支持向量机、SMO
实践案例: 使用SVM进行数据分类 

第13课 贝叶斯网络
朴素贝叶斯、有向分离、马尔科夫模型/HMM/pLSA

第14课 EM算法
GMM、pLSA、HMM
实践案例：分解男女身高、图像分割

第15课 主题模型
pLSA、共轭先验分布、LDA
实践案例：使用LDA进行文档聚类 

第16课 采样与变分
MCMC/KL(p||q)与KL(q||p)

第17课 隐马尔科夫模型HMM
概率计算问题、参数学习问题、状态预测问题
实践案例：使用HMM进行中文分词 

第18课 条件随机场CRF
概率无向图模型、MRF、线性链CRF

第19课 人工神经网络
BP算法、CNN、RNN

第20次课 深度学习
实践案例：使用Torch进行图像分类及卷积网络可视化的深度学习实践


预习

《高等数学·上下册》；
《概率论与数理统计·浙大版》、《数理统计学简史·陈希孺》；
《矩阵分析与应用·张贤达》；
《凸优化(Convex Optimization) · Stephen Boyd & Lieven Vandenberghe著》；
《统计学习方法·李航》；
《Pattern Recognition And Machine Learning · Christopher M. Bishop著》，简称PRML；
