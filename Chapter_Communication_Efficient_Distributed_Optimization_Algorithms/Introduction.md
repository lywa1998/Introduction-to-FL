# 概述

**ML训练中的随机梯度下降。**大多数监督学习问题都能使用经验风险最小化框架来解决[5, 41]，其目标是最小化经验风险目标函数$F(\bm{x}) = \sum_{j=1}^{n}f(\bm{x}; \xi_{j})/n$。其中，$n$是训练数据集的大小，$\xi_{j}$是第$j$个标记的训练样本，$f(\bm{x}; \xi_{j})$是（通常是非凸的）损失函数。优化$F(\bm{x})$的普遍算法是随机梯度下降法(SGD)，在这个算法中，我们计算$f(\bm{x}; \xi_{n})$在小的、随机选择的子集$\mathcal{B}$（称为批量）上的梯度，每个子集有$b$个样本[4, 12, 25, 35, 37, 53]，并根据$\bm{x}_{k+1} = \bm{x}_{k} - \eta \sum_{i \in \mathcal{B}} \nabla f(\bm{x}_{k}; \xi_{i})$来更新$\bm{x}$，其中$\eta$被称为学习率或步长。虽然是为凸目标而设计的，但由于其能够摆脱鞍点和局部最小值，小型批量SGD甚至在非凸损失表面上也表现良好[7, 31, 42, 55]。因此，它是最先进的机器学习中的主导训练算法。

对于像Imagenet[38]这样的大规模数据集，在单个节点上运行小批量的SGD会慢得令人望而却步。梯度计算并行化的一个标准方法是参数服务器（PS）框架[11]，由一个中央服务器和多个工作节点组成。在将这个框架扩展到大量的工作节点时，交错的工人和通信延迟会成为一个瓶颈。一些方法，如异步[10, 13, 15, 56]和周期性梯度聚合[43, 46, 54]已被提出，以提高基于数据中心的ML训练的可扩展性。

**联合学习的动机。**尽管算法和系统的进步提高了效率和可扩展性，但基于数据中心的训练有一个主要限制。它要求训练数据集集中在参数服务器上，而参数服务器会在工作节点上进行洗牌和拆分。边缘方的迅速扩散，如手机、物联网传感器和具有设备上计算能力的相机，导致了这种数据分割范式的重大转变。边缘方从他们的环境中收集丰富的信息，这些信息可用于数据驱动的决策。由于有限的通信能力以及隐私问题，这些数据不能直接发送到云端进行集中处理或与其他节点共享。联合学习框架建议将数据留在边缘方，而将模型训练带到边缘。在联合学习中，数据被保存在边缘方，模型以分布式的方式被训练。只有梯度或模型更新在边缘方和聚合器之间进行交换。

**系统模型和符号。**如图6.1所示，一个典型的联合学习环境包括一个连接到$K$个边缘方的中央聚合器，其中$K$可能是数千甚至数百万的数量级。每一方$i$都有一个由$n_{i}$个样本组成的本地数据集$\mathcal{D}_{i}$，它不能被转移到中央聚合器或与其他边缘方共享。我们使用$p_{i} = n_{i}/n$来表示第$i$方的数据部分，其中$n 
 = \sum_{i=1}^{K}n_{i}$。聚合器试图利用本地数据集$\mathcal{D} = \cup_{i=1}^{K}\mathcal{D}_{i}$的联盟来训练一个机器学习模型$\bm{x} \in \mathbb{R}^{d}$。模型向量$\bm{x}$包含模型的参数，例如，神经网络的权重和偏差。为了训练模型$\bm{x}$，聚合器寻求最小化以下经验风险目标函数：
$$F(\bm{x}) := \sum_{i=1}^{K}p_{i}F_{i}(\bm{x})$$
其中$F_{i}(\bm{x}) = \frac{1}{n_{i}} \sum_{\xi \in \mathcal{D}_{i}} f(\bm{x}; \xi)$是第$i$方的本地目标函数。这里，$f$是由模型$\bm{x}$定义的损失函数（可能是非凸的），$\xi$代表本地数据集$\mathcal{D}_{i}$的数据样本。请注意，我们分配的权重与第$i$方的数据分数成正比。这是因为，我们希望模拟一个集中式的训练场景，将所有的训练数据传输到一个中央参数服务器。因此，拥有更多数据的一方将在全局目标函数中获得更高的权重。

由于边缘方的资源限制和大量缔约方的存在，联合训练算法必须在严格的通信限制下运行，并应对数据和计算的异质性。例如，连接每个边缘方和中央聚合器的无线通信链路可能有带宽限制，并且有很高的网络延时。另外，由于有限的网络连接和电池限制，边缘方可能只是间歇性地可用。因此，在给定的时间里，K个边缘方中只有m个子集可以参与模型x的训练。为了在这些通信约束条件下运行，联合学习框架需要新的分布式训练算法，超越在数据中心环境中使用的算法。在第6.2节中，我们回顾了本地更新的SGD算法及其变体，这些算法减少了边缘各方与聚合器的通信频率。在第6.3节中，我们回顾了压缩和量化的分布式训练算法，这些算法减少了边缘方发送给聚合器的每次更新的比特数。