确定性能度量和目标后，任何实际应用的下一步是尽快建立一个合理的端到端的系统。
本节给出了一些关于在不同情况下使用哪种算法作为第一个基准方法推荐。
在本节中，我们提供了关于不同情况下使用哪种算法作为第一基准方法的推荐。
值得注意的是，深度学习研究进展迅速，所以本书出版后很快可能会有更好的默认算法。
<!-- % 413 head  -->

# 选择一类合适的模型  

> **[success]**  
> 简单的统计模型，全连接的前馈网络，CNN，RNN  

根据问题的复杂性，项目开始时可能无需使用深度学习。
如果只需正确地选择几个线性权重就可能解决问题，那么项目可以开始于一个简单的统计模型，如逻辑回归。


如果问题属于"AI-完全"类的，如对象识别、语音识别、机器翻译等等，那么项目开始于一个合适的深度学习模型，效果会比较好。
<!-- % 413 mid  -->


首先，根据数据的结构选择一类合适的模型。
如果项目是以固定大小的向量作为输入的监督学习，那么可以使用全连接的前馈网络。
如果输入有已知的拓扑结构（例如，输入是图像），那么可以使用卷积网络。
在这些情况下，刚开始可以使用某些分段线性单元（ReLU\,或者其扩展，如\,Leaky ReLU、PReLU\,和\,maxout）。
如果输入或输出是一个序列，可以使用门控循环网络（LSTM\,或\,GRU）。
<!-- % 413 mid  -->

# 优化算法  

> **[success]**  
> 具有衰减的学习率  
> 带动量的SGD  
> 批标准化  

具有衰减学习率以及动量的\,SGD\,是优化算法一个合理的选择
（流行的衰减方法有，衰减到固定最低学习率的线性衰减、指数衰减，或每次发生验证错误停滞时将学习率降低$2-10$倍，这些衰减方法在不同问题上好坏不一）。
另一个非常合理的选择是Adam算法。
批标准化对优化性能有着显著的影响，特别是对卷积网络和具有~sigmoid~非线性函数的网络而言。
虽然在最初的基准中忽略批标准化是合理的，然而当优化似乎出现问题时，应该立刻使用批标准化。
<!-- % 413 mid  -->

# 正则化  

> **[success]**  
> 提前终止  
> dropout  

除非训练集包含数千万以及更多的样本，否则项目应该在一开始就包含一些温和的正则化。 
提前终止也被普遍采用。
Dropout~也是一个很容易实现，且兼容很多模型和训练算法的出色正则化项。
批标准化有时也能降低泛化误差，此时可以省略~Dropout~步骤，因为用于标准化变量的统计量估计本身就存在噪声。 %?? 还是有问题
<!-- % -- 413 --  end -->

# 复制一个已有的模型  

如果我们的任务和另一个被广泛研究的任务相似，那么通过复制先前研究中已知性能良好的模型和算法，可能会得到很好的效果。
甚至可以从该任务中复制一个训练好的模型。
例如，通常会使用在ImageNet上训练好的卷积网络的特征来解决其他计算机视觉任务{cite?}。
<!-- % 414 head -->

# 无监督学习  

一个常见问题是项目开始时是否使用无监督学习，我们将在第三部分进一步探讨这个问题。
 这个问题和特定领域有关。
在某些领域，比如自然语言处理，能够大大受益于无监督学习技术，如学习无监督词嵌入。
在其他领域，如计算机视觉，除非是在半监督的设定下（标注样本数量很少）{cite?}，目前无监督学习并没有带来益处。
如果应用所在环境中，无监督学习被认为是很重要的，那么将其包含在第一个端到端的基准中。
否则，只有在解决无监督问题时，才会第一次尝试时使用无监督学习。
在发现初始基准过拟合的时候，我们可以尝试加入无监督学习。