# -Crop-classification
-----------------------------------------
# 目录
-  [1. 赛题要求](#1-赛题要求)
-  [2. 方法与模型](#2-方法与模型)
    -   [方法1](#方法1)
    -   [方法2](#方法2)
    -   [方法3](#方法3)
    -   [方法4](#方法4)

# 1. 赛题要求
 ------------------------------------------
一句话介绍：百度第二届高分杯美丽乡村大赛。  在有少量样本的情况下，对含有三种农作物和背景的一幅多光谱图像进行农作物分类

* 题目介绍

 赛事数据来源于某一时刻一张遥感卫星多光谱图像，覆盖850公里*300公里。要求参赛队利用深度学习等智能算法自动识别出所给图像对应的农作物，包括玉米、大豆、水稻三种农作物区块和其他区块共四种区块，根据参赛团队对场景的识别准确度和时效性进行评分。

* 数据简介

 赛事将提供遥感卫星图像（格式为tif）； 

* 数据类别

 赛事数据来源于某一时刻一张遥感卫星的历史存档数据，是多光谱图像，覆盖850公里*300公里。

* 数据内容

 赛事数据包括：数据内容覆盖玉米、大豆、水稻和其它区块等4类典型农作物场景；玉米、大豆和水稻3类农作物分布在遥感卫星图像中不同区块，不同农作物区块数量不一，不同区块面积大小不一；除以上三类的农作物以外的图像区域定义为其它区块。

 数据格式为多光谱tif图像。赛事目标是从测试数据集中识别出玉米、大豆和水稻所在区块，并把对应的像素点值标注为对应类别的农作物类别值。

* 数据组成

 本次竞赛的数据由原始多光谱图像和训练数据集两部分组成：
 
  原始多光谱图像：

  多光谱图像一张（tif格式），8通道，覆盖面积850公里*300公里

  训练数据集：

  训练数据集是原始多光谱图像中农作物区块的部分标注数据。

  标注样本点数据，给定玉米、大豆和水稻3个类别农作物对应区块的中心点像素位置（x,y）列表，以及对应中心点对应的区块半径3

```

    样例：FID,Id,作物,半径,备注,x,y

          0,1,玉米,3, ,12500.7001953,-3286.5600586

          1865,1866,大豆,3, ,5941.6601563,-6966.2797852

          2086,2087,水稻,3, ,9165.4697266,-14989.2998047
          
```


* 备注：

  1、（x，y）为tif数据格式的坐标系；（x,y）取值为小数，选手可以四舍五入取整获得对应的像素点位置；选手可以从卫星图片中取出对应图片7*7*8作为训练样本；某些农作物区块对应的面积半径可能大于3，选手可以用算法扩展农作物区块面积，作为训练样本；

  2、多光谱图像由多张高分图像拼接而成，会有光照等影响因素使得同类农作物区块的颜色可能不同，此处考验的是选手所训练模型的泛化能力。

  数据对应关系：RasterXSize对应的是x坐标，RasterYSize对应的是y坐标，x坐标步长是1.0，y坐标步长是-1.0，左上角坐标是(0.0, 1e-07)
  
# 2. 方法与模型
 -------------------------------------------
  - ### 方法1
    采用无监督的学习方法，利用自编码器学习样本特征，结合余弦相似度求解
  - ### 方法2
    采用无监督与有监督结合的方法，将自编码器学习到的特征，结合SVM进行分类
  - ### 方法3
    采用自训练的方法[self-training](#self-training)，扩展样本集，训练分类器，进行分类
  - ### 方法4
    采用协同训练的方法[co-training](#co-training)，扩展样本集，训练分类器，进行分类
    
  ### self-training
 
   自学习模型的基本假设是，分类器对样本进行预测时，置信度高的样本被正确分类的可能性大。比如SVM对某个样本进行分类时，离分类界面距离较远的那些样本可以认为被正确分类。那么基于这个假设，自学习模型就显得异常简单了。假设我有两堆数据A和B，其中A是已标注的数据，即带Label的；而B是未标注数据。Self-Training的做法如下：


```
    - 从已标注数据A中训练一个分类模型M
    - 用该模型对B进行预测
    - 将预测结果中置信度高的K个样本，连同它们的Label加入训练数据A，并从B中删除
    - 回到第1步。
    - 当然，第3步可以有很多实现方法，比如可以将B中所有样本加入A，根据预测时置信度的不同给样本赋予不同的权重。
```

   自学习模型是最简单也最易实现的半指导模型，但是简单的东西往往不怎么好用，这世上既简单又好用的东西实在不多。自学习的缺点在于，如果一个错误分类的样本被加入了原来的训练集，那么在其后的训练过程中，它所犯的错误只会越来越深，还会诱使其他样本犯错，这种outlier实在罪不容恕。而自学习模型对此表示无能为力。
    
    
  ### co-training
 
   Co-Training又叫协同训练或协同学习，是一种MultiView算法。Multiview是指认识事物的多个角度。比如对于“月亮”，我们会在脑海里浮现出一轮飞镜似的明月，或圆或缺，或明或暗，我们甚至还会联想到古代词人吟风弄月的很多佳句。当然也有人会立刻联想到月亮的各种特征，比如它是地球的卫星，它的表面有环形山，它绕地运行……等等一系列特征。这便是看待同一件事物的两个角度。那么从不同的角度看待训练数据，我们能够得到不同的特征空间。而在不同的特征空间中，我们又能够得到不同的分类模型。这就是Co-Training的基本思想。

   协同训练的过程如下：假设数据有两种特征表达，比如图像特征（X-1, Y-1）和文本特征（X-2, Y-2）。对于未标注数据同样有两种View。算法如下：
```
    - 从（X-1, Y-1），（X-2, Y-2）分别训练得到两个个分类模型F-1，F-2
    - 分别使用F-1与F-2对未标注数据进行预测
    - 将F-1所预测的前K个置信度最高的样本加入F-2的训练数据集
    - 将F-2所预测的前K个置信度最高的样本加入F-1的训练数据集
    - 回到第1步
```

   基本的Co-Training算法还是很简单的。如何将其应用到NLP的Task里又有很多研究点。我觉得从Co-Training的思想出发可以稍微帮助理解一下人工智能。一直以为人工智能一定是群体作用的结果，一个封闭的自学习系统是很难激发出智能的，除非本身拥有一个非常强大的知识集和归纳、演绎系统。一个智能体，在幼儿时主要依靠专家指导（我们的父母亲人）来强化自身的学习系统（人脑神经网络），在学习系统成长的过程中，我们也在不断地和我们的同伴相互对比，相互学习。在这不断地碰撞和启发中，我们才渐渐地拥有了知识和对事物准确的判断力。人的这种获取智能的方式应该是值得机器学习方法所借鉴的。或许有一天，我们真的能够模拟人脑的思维，而不再仅仅是模拟人类的行为。
        
     
   > 参考文献 http://www.leexiang.com/self-training-and-co-training
