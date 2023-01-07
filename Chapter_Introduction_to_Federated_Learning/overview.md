# 1.1 概述

机器学习（ML）已经成为开发认知和分析功能的关键技术，而这些功能在算法上很难得到有效的开发。随着深度神经网络（DNN）和有效训练复杂网络的计算硬件的出现，计算机视觉、语音识别和自然语言理解方面的应用取得了飞跃性的进展。此外，经典的ML技术，如决策树、线性回归和支持向量模型（SVMs）也得到了更多的应用，特别是与结构化数据有关的应用。

ML的应用在很大程度上取决于高质量训练数据的可用性。但有时，隐私方面的考虑使训练数据无法被带到一个中央数据存储库中，以便为ML过程进行策划和管理。联合学习（FL）是在[28]中首次以这个名字提出的一种方法，在不同地点的训练数据上训练ML模型，不需要集中收集数据。

这种不愿意使用中央数据存储库的一个重要驱动因素是不同司法管辖区的消费者隐私法规。欧盟的《一般数据保护条例》（GDPR）[50]、《健康保险可携性和责任法案》（HIPAA）[53]和《加州消费者隐私法案》（CCPA）[48]是收集和使用消费者数据的监管框架的范例。此外，关于数据泄露的新闻报道提高了人们对存储敏感消费者数据所带来的责任的认识[9, 42, 43, 51]。FL为使用数据提供了便利，而实际上不需要将其存储在一个中央存储库中，从而减轻了这种风险。监管也限制了数据在不同国家等管辖区之间的流动。这是因为考虑到其他国家的数据保护可能不足或与国家安全有关，要求关键数据保留在岸上[40]。国家和地区的法规对在不同市场拥有子公司但希望使用其所有数据来训练模型的国际公司构成了挑战。除了监管要求，从不同地点的数据中学习也可能只是实用。糟糕的通信连接和由传感器或电信设备收集的大量数据会使中央数据收集不可行。FL也使不同的公司能够在不泄露其商业秘密的情况下，共同创建互利的模型。

那么FL是如何工作的呢？在FL方法中，一组控制着各自训练数据的不同各方，合作训练一个机器学习模型。他们这样做并不与其他各方或任何其他第三方实体分享他们的训练数据。合作的各方在文献中也被称为客户或设备。当事人可以是各种各样的东西，包括消费者设备，如智能手机或汽车，但也包括不同供应商的云服务，在不同国家处理企业数据的数据中心，公司内部的应用仓，或嵌入式系统，如汽车厂的制造机器人。

虽然FL协作可以以不同的方式进行，但其最常见的形式概述于图1.1。在这种方法中，一个聚合器，有时被称为服务器或协调器，促进了合作。各方根据他们的私人训练数据进行本地训练。当他们的本地训练完成后，他们将他们的模型参数作为模型更新发送到聚合器。模型更新的类型取决于要训练的机器学习模型的类型；例如，对于一个神经网络，模型更新可能是网络的权重。
一旦聚合器收到各方的模型更新，它们就可以被合并到一个共同的模型中，这个过程我们称之为模型融合。在神经网络的例子中，这可以像FedAvg算法[38]中提出的那样，简单地对权重进行平均化。然后，合并后的模型将作为模型更新再次分发给各方，以形成下一轮学习的基础。这个过程可以重复多轮，直到训练过程收敛。
聚合器的作用是协调各方的学习过程和信息交流，并执行融合算法，将各方的模型参数合并为一个共同的模型。融合过程的结果是一个基于各方训练数据的模型，而训练数据从不共享。

FL方法似乎与集群上的分布式学习有关[15]，这是大型ML任务的一种常见方法。分布式学习使用一个计算节点集群来分担ML的计算工作，从而加速学习过程。分布式学习通常使用一个参数服务器来汇总各节点的结果，这与联合模型中并无不同。然而，它在一些重要方面是不同的。在FL中，数据的分布和数量不是集中控制的，如果所有的训练数据都是私有的，可能是未知的。我们不能对各方数据的独立性和相同分布（IID）做出假设。同样地，一些当事方可能比其他当事方拥有更多的数据，导致当事方之间数据集的不平衡。在分布式学习中，数据被集中管理，并以分片形式分布到不同的节点，中央实体了解数据的随机属性。在设计FL训练算法时，必须考虑到各方数据的不平衡性和非IID性。

相反，在FL中，各方的数量可能会有所不同，这取决于用例。
在一家跨国公司的不同数据中心的数据集上训练一个模型，可能有少于10个当事人。这通常被称为企业[35]或跨语境用例[26]。在一个移动电话应用的数据上进行训练，可能会有数以亿计的各方贡献。这通常被称为跨设备用例[26]。在企业用例中，一般来说，在每一轮中考虑所有或大多数当事方的模型更新是很重要的。在设备用例中，每一轮FL只包括全部设备中的一个潜在的大子样本。在企业用例中，FL过程考虑了相关各方的身份，并可以在训练和验证过程中使用这些。在跨设备的使用案例中，当事人的身份通常并不重要，而且一个当事人可能只参与一轮训练。

在设备用例中，比起企业场景，考虑到参与者的数量众多，可以假设一些设备的通信失败。手机可能关闭，或者设备可能处于网络覆盖不佳的地区。这可以通过采样方和设置时间限制来执行聚合，或其他缓解技术来管理。在企业用例中，由于参与者人数不多，个人的贡献是相关的，所以必须仔细管理通信故障。

在本章的其余部分，我们将对FL进行概述。我们在下一节中对所使用的主要概念进行了正式介绍。之后，我们从三个重要的角度讨论FL，每个角度都有一个单独的章节。首先，我们从机器学习的角度讨论FL；然后，我们通过概述威胁和缓解技术来涵盖安全和隐私的角度；最后，我们对联合学习的系统角度进行了概述。这将为本书的其余部分提供一个起点。