---
layout: post
title: 学生论文分享第四期
icon: star-o
---

分享内容：  
1.赵子豪：《Experimental Analysis of Distributed Graph Systems》  
2.Fayaz Ali： 《A Review of Researches on Deep Learning in Remote Sensing Application》   
3.乔子越：《Gated Graph Sequence Neural Networks》 ICLR’16  

# 附件
[Graph System.pptx.(赵子豪)](/img/presentation/2019-03-18/2019.3.18Graph System.pptx)  
[Weekly_PPT.1.pptx.(Fayaz Ali)](/img/presentation/2019-03-18/Weekly_PPT.1.pptx)  
[GGNN.pptx.(乔子越)](/img/presentation/2019-03-18/GGNN.pptx)

# 1.《Experimental Analysis of Distributed Graph Systems》
## Abstract
#### Graph processing systems:
1. Hadoop
2. HaLoop
3. Vertica
4. Giraph
5. GraphLab (PowerGraph)
6. Blogel
7. Flink Gelly
8. GraphX (SPARK)

#### Workloads:
1. PageRank
2. WCC
3. SSSP
4. K-hop


#### Dataset:
1. Twitter
2. World Road Network
3. UK 200705
4. ClueWeb
5. 


## 1.Introduction
针对于大规模图数据，有许多解决方案，大多数是分布式并行系统，也有少部分单机解决方案。本文分析的对象专注于分布式系统。


-  Vertex-centric:Synchronous: Giraph [3], GraphLab [23], BlogelVertex (Blogel-V) [46]


## 2.Systems
### 2.1 Vertex-Centric BSP
Each vertex computes its new state based on its current state
and the messages it receives from its neighbors. Each vertex then sends its new state to its neighbors using message
passing. Synchronous versions follow the Bulk Synchronous
Parallel (BSP) model that performs parallel computations
in iterative steps, and synchronizes among machines at the
end of each step. This means that messages sent in one iteration are not accessible by recipients in the same iteration; The computation stops when all vertices converge to a fixpoint or after a predefined number of iterations. 

#### 2.1.1 Giraph
 Giraph is implemented as a map-only application on Hadoop. It requires all data to be loaded in memory before starting the execution.  Graph data is partitioned randomly using edge-cut approach, and each vertex is assigned to a partition. 
 
#### 2.1.2 GraphLab/PowerGraph
GraphLab is a distributed graph processing system
that is written in C++ and uses MPI for communication.
Similar to Giraph, it keeps the graph in memory.

It has three functions: Gather, Apply, and Scatter (GAS). The
GAS model allows each vertex to gather data from its
neighbors, apply the compute function on itself, and
then scatter relevant information to some neighbors if
necessary.

 - vertex-cut

 This replicates vertices and helps
better distribute the work of vertices with very large
degrees. 

#### Blogel-V
Blogel adopts both vertex-centric and block-centric
models.

Blogel is implemented in C++ and uses MPI for communication between nodes. 


### 2.2 Vertex-Centric Asynchronous
GraphLab支持一种异步模式，使得再一次迭代中，节点就可以获取邻居节点的最新数据。不用等一次迭代全部完成才开始新的一次迭代。通过分布式加锁实现的。

### 2.3 Block-Centric BSP
也叫graph-centric。主要思想是将图分割成顶点块，使用BSP在单独的机器上阻塞，同步时在块内运行串行算法。目标是减少迭代次数。减少了网络通信时间。Giraph++和Blogel都是这种方法。

Blogel-B: 对一个block有一个serial graph algorithm。使用GVD（Graph Voronoi Diagram）把数据集分成多个连通子图。

### 2.4 MapReduce
提供Map和Reduce两个接口，极大便利并行处理。
Hadoop是最有名的开源实现。因为I/O开销大，还有shuffling的耗时大，Hadoop不适合用迭代轮次多的图算法。但是可以用于内存不够的场景中。

### 2.5 MapReduce Optimized
在map和reduce步骤之间缓存可以复用的数据，避免对无关数据的重复扫描以及在机器之间的shuffling操作。

#### 2.5.1 HaLoop
主要目的是减少数据shuffling和reduce工作，在迭代负载上做出如下改进以提高性能：
1. 适合迭代变成的模型：比如在主节点增加循环控制
2. 改进任务调度器，减少网络通信。
3. 从节点上缓存一些loop-invariant的数据并建立索引。
4. 最后一轮迭代的结果缓存在本地，用于后续比较，无需去HDFS里边取回。

#### 2.5.2 Spark/GraphX
在内存中划分数据集。RDD。使用节点划分，每一轮迭代包含多个Spark jobs。

### 2.6 Relational
Vertica，这种系统使用关系型数据库存储数据（数据库实现了分布式存储）。重复的join操作效率低，vertica做出如下改进：
1. 在vertex table上更新多个值开销比创建新表更大。超过阈值就创建新表，而非修改值。
2. 在一些遍历类的任务（SSSP）中每轮迭代只用一部分数据，将活跃点保存在一个临时的table中。


### 2.7 Stream Systems
定义操作，形成拓扑图，数据在拓扑图中被处理、传递并形成数据流。
本文分析了Flink Gelly。两种approach：stream和batch。为了保证和其他工具一致，文中使用的是batch，这样可以把准备数据的时间和计算时间分开。

## 3. Workloads

### 3.1 PageRank
### 3.2 WCC
weakly connected component
### 3.3 SSSP
Single Source Shortest Path
### 3.4 K-hop
使用随机起点。


## 4. Experiment Design

Dimension | Values
---|---
Systems | Giraph, Blogel, Hadoop, HaLoop, GraphX, GraphLab, Vertica, Flink Gelly
Workloads | WCC, PageRank, SSSP, K-hop
Datasets  | Twitter, UK, ClueWeb, WRN
Cluster Size | 16, 32, 64, 128

### 4.1 Infrastructure
Amazon EC2 AWS r3.xlarge machine.

4核s, 30.5G内存, Xeon E5-2679 v2, SSD硬盘。

单节点，16、32、64、126台机器。

### 4.2 Evaluated Metrics
评估两项指标：资源利用率和系统性能。

记录如下性能度量：
1. 数据加载时间
2. 结果保存时间
3. 执行时间
4. 总体响应时间

（理想情况下1+2+3=4，但实际4中包括分片时间，网络通信时间，同步时间）

### 4.3 Datasets

Dataset | \|E\| |Avg./Max. Degree| Diameter
:---:|:---:|:---:|:---:
Twitter | 1.46B | 35 / 2.9M | 5.29
WRN | 717M | 1.05 / 9 | 48K
UK200705 | 3.7B | 35.3 / 975k | 22.78
ClueWeb  | 42.5B | 43.5 / 75M | 15.7

## 5. Result&Analysis

几种错误：
- TO：超时错误，24h
- OOM：内存溢出
- MPI：MPI错误（仅发生在Blogel-B）
- SHFL:shuffle错误，仅发生在HaLoop中


### 5.1 Blogel：The Overall Winner
端到端的性能最好
使用WRN数据集，唯一在所有节点上都完成了SSSP和WCC测试的系统。

唯一可以在128台机器的集群上完成ClueWeb计算的。

因为没有昂贵的“infrasturcture”（such as Hadoop or Spark）。使用高效的C++库，优化利用所以CPU核，有小的内存footprint。

在一些可达性的查询上（SSSP,WCC,K-hop等）有最好的表现，因为：
1. 这些查询从Voronoi划分上收益
2. 基于Block的计算最小化了网络通信的开销
3. 

BB:在一些可达性的查询上（SSSP,WCC,K-hop等）有最好的表现，因为：这些查询从Voronoi划分上收益,基于Block的计算最小化了网络通信的开销，而且降低了网络通信

BB运行PageRank的表现不好，因为在block上跑是为了给全局计算提供一个好的初始值，但是这种算法没有产生好的初始值，导致全集性能降低，导致需要更多次迭代。

### 5.2 Exact vs. Approximate PageRank
exact假设每轮迭代，每个节点都参与计算。近似算法允许PR值不发生变化。
GraphLab是唯一一个实现了后者的系统，因为活跃节点可以从不活跃的邻居节点收集信息。但是这也增大了内存消耗，所以GraphLab发生了内存错误。

### 5.3 Syncchronous vs. Asychronous
GraphLab是唯一一个提供了异步计算模式的系统。但是在每台worker节点上开了数以千计的新进程处理节点，导致分布式加锁争用问题。因此PageRank的异步计算比同步计算更慢。还发现异步计算只对特殊规模的集群适用。具体规模需要尝试。


### 5.6 GraphX is not efficient when large number of iterations are required.


GRAPHX/SPARK比所有其他系统都慢。因为spark overhead，data shuffle,长的RDD血统，和checkpoint。以前的研究表明，GRAPHX高效，因为它使用了一个特殊的Spark原型版本，in-Memory的shuffle。此功能在最新版本不可用
GraphX在所有规模的集群上基于WRN做WCC都失败了，因为超过了内存大小，有的是超时。结果证明，spark保持RDD血统的容错机制导致了内存错误。当迭代次数增加时，这些lineage就变成了内存占用的大户，导致了潜在的内存不足错误。

最近引入的Graphframes[11]对于graphx应该是一个更有效的选择。我们研究了Graphframes的实现，发现它的许多算法可以将输入的图转换为GraphX格式，然后运行GraphX算法。我们还发现，大多数算法对迭代次数有一个默认的最大限制，以减少RDD中长lineage的潜在开销。例如，sssp的限制为10，否则它将开始进入一个checkpoint。此外，WCC的实现要求每两轮迭代就进行一次检查点。检查点可以避免lineage变得太长，但是导致了昂贵的I/O开销，着就会导致超时错误。GraphFrames提供了一个hash-min算法去计算WCC，这种方法迭代次数少，经测试与Blogel性能差不多。

还注意到，spark不能在节点之间做负载均衡，一些机器分的partition很多，在同步计算模型，最慢的节点拖慢了整体进程。在实际情况中，花分数不应该超过输入文件中的block数，因为这会导致spark多次读同一个data block。在另一方面，划分数也不应该比集群中的核数少，这导致CPU使用率低。

# 2. 《A Review of Researches on Deep Learning in Remote Sensing Application》

## Introduction:


### 1.  Understanding Images:
As Images are the lagest source of data so we need to understand in order to solve major problems.such as sheer size of data/ 
The problems of search, annotation, and classification of videos on Youtube are examples of challenges where the size of image data becomes a problem.

Directed vs. Undirected Graphical Models
There are two types of graphical models: Directed Graphical Model (or Directed Acyclic Graphs- DAG)
and Undirected Graphical Model (UGM). The directed edges in a DAG give causality relationships, DAGs
are also called Bayesian Network. The undirected edges in UGM give correlations between variables,
UGMs are also called Markov Random Fields (MRF). There are two types of ways to organize and represent relationships between variables. Here is an example of representing the regulatory relationships of proteins and genes in both ways.

### 2. Remote sensing: 
It is a technical means using sensors on satellite, aircraft or other platforms to collect targets’ radiation information, with which specific information can be obtained. In recent years, with the rapid development of remote sensing technology, the capacity of acquiring remote sensing data has been enhancing. Meantime, the spectral, spatial and temporal resolution of remote sensing imagery have been improving.

Deep learning is an important domain of machine learning research. Compared with traditional machine learning, deep learning is a representationlearning method with multiple layers. Data abstraction and extraction from the
lower layers to higher layers are accomplished through simple nonlinear modules. Current deep learning often use deep neural network (DNN) to construct the layers, which are the stacks of simple nonlinear modules. Input data is passed
between the layers, whose mapping relationship reduces the dimension and extract the key characteristics of data [4]. Relying on the deep convolution neural network (DCNN), deep learning provides an end-to-end machine learning
model that can automatically extract image features without extraction algorithms designed by human. Compared with traditional methods, deep learning
is completely data-driven, which can automatically find the best ways to extract image features through learning.

#### 2.1 Common Deep Learning Methods in Remote Sensing Application:
the deep learning method in remote sensing application is mainly used in three
aspects, namely surface classification, object detection and change detection. A
review of the current research results indicates that the major technical approachM is to translate specific problems into classification or object detection tasks, which are processed with the computer vision deep learning model that is redesigned and adjusted for the targets of the remote sensing application, thus the specific problems are solved. 

1. surface classification:   
Land cover classification is a major field of remote sensing application. The
main task of surface classification is to divide the pixels or regions in remote
sensing imagery into several categories according to application requirements
[4,7]. The deep learning model of land cover classification is generally based on
deep belief network (DBN), convolution neural network (CNN) and spare auto
encoder (SAE), among which the deep convolution neural network is the most
popular approach at present.  
Many early studies used deep CNN as Alexnet and VGG Net and achieved certain results. However, the nature of Alexnet and VGG Net classification method is to transform an image into a corresponding eigenvector through convolution, pooling and fully connected layer. Based on the eigenvector, a value representing the image classification is output. Therefore, the major issue addressed with such
approach is the classification of integrated imagery on the image level. However, land cover classification is a problem of image segmentation, what to be addressed is the multi-classification after semantic segmentation of a single image

2. object detection:  
Object detection is another common application of remote sensing. The deep
learning model of object detection is mainly based on region-based convolution
neural networks (R-CNN), which is the earliest proposed method of deep learning object detection. The main idea is to transform the object detection problem
into the classification problem. The image is divided into a large number of candidate regions by selective search algorithm, CNN is then applied to obtain the
eigenvectors of candidate regions, and finally object detection is completed by
the classifier, which determines the type of the candidate area [7]. The proposal
of R-CNN has greatly improved the success rate of image object detection, but
R-CNN will generate partially overlapping candidate areas from each detection
target. Such areas are repeatedly fed into CNN for feature calculation, thus reducing the efficiency of detection. To reduce overlapping candidate areas, He
Kaiming proposed Spatial Pyramid Pooling Networks (SPP-Net) [14], which introduces the spatial pyramid pooling layer after the last convolution layer, thus
repetitive processing is eliminated, allowing image of any sizes to be processed
with CNN. With these improvement, SPP-Net has greatly increased the speed of
object detection. Based on SPP-Net, Girshick proposed Fast R-CNN [15], which
simplifies the spatial pyramid pooling layer of SPP-Net, thus, the RoI pooling
layer is formed to extract features. The substitution of SVM by Softmax greatly
improves the speed of training and detection. It is more accurate and 213 times
faster than R-CNN. To further improve the efficiency of Fast R-CNN in generating candidate area, Ren et al. proposed Faster R-CNN [16], which introduces Region Proposal Network (RPN), meantime, RPN and Fast R-CNN are combined as an integrated network to generate candidate regions. With further improved network structure, YOLO [17] and Single Shot Multibox Detector (SSD)
[18] maintain almost the same detection accuracy with significantly improved
detection speed
 
3. change detection 
Change detection is the process of detecting changes using remote sensing imagery obtained at different times. These changes are due in part to natural phenomena, such as droughts, floods, and landslides, the other part is due in human
activities as new roads, excavation of the surface or construction of new houses.
Compared to models for surface classification and object detection, there are less
deep learning models for image change detection [7]. The current change detection based on deep learning mainly adopts two technic approaches. One is to detect the correspondent points of two imagery through deep learning and determine whether there are changes to the correspondent points. The other approach is to translate the change detection problem into the surface classification
problem, and acquire the changed region through semantic segmentation, comparing and classification of map spots. From the experimental results, the semantic segmentation approach is easier to achieve, faster in speed and better in
detection accuracy.

---
# Gated Graph Sequence Neural Networks
[《Gated Graph Sequence Neural Networks》阅读笔记, 孙建东](https://zhuanlan.zhihu.com/p/28170197)


# Reference:
[1] Li, D.R., Tong, Q.X., Li, R.X., Gong, J.Y. and Zhang, L.P. (2012) Current Issues in
High-Resolution Earth Observation Technology. Science China Earth Sciences, 55,
1043-1051. https://doi.org/10.1007/s11430-012-4445-9  
[2] Pagot, E. and Pesaresi, M. (2008) Systematic Study of the Urban Postconflict
Change Classification Performance Using Spectral and Structural Features in a
Support Vector Machine. IEEE Journal of Selected Topics in Applied Earth Observations & Remote Sensing, 1, 120-128.
https://doi.org/10.1109/JSTARS.2008.2001154  
[3] Li, D.R., Zhang, L.P. and Xia, G.S. (2014) Automatic Analysis and Mining of Remote Sensing Big Data. Acta Geodaetica et Cartographica Sinica, 43, 1211-121  
[4] Lecun, Y., Bengio, Y. and Hinton, G. (2015) Deep Learning. Nature, 521, 436-444.
https://doi.org/10.1038/nature14539  
[5] Gong, J.Y. and Ji, S.P. (2018) Photogrammetry and Deep Learning. Acta Geodaetica
et Cartographica Sinica, 47, 693-704.  
[6] Gong, J.Y. and Ji, S.P. (2017) From Photogrammetry to Computer Vision. Geomatics and Information Science of Wuhan University, 42, 1518-1522.  
[7] Ball, J.E., Anderson, D.T. and Chan, C.S. (2017) A Comprehensive Survey of Deep
Learning in Remote Sensing: Theories, Tools, and Challenges for the Community.
Journal of Applied Remote Sensing, 11. https://doi.org/10.1117/1.JRS.11.042609  
[8] Badrinarayanan, V., Kendall, A. and Cipolla, R. (2017) SegNet: A Deep Convolutional Encoder-Decoder Architecture for Image Segmentation. IEEE Transactionson Pattern Analysis and Machine Intelligence, 39, 2481-2495.  
