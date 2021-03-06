收集数据刚刚比改进算法有用  

# 决定是否收集更多的数据

训练集上效果差：没必要，增加模型的规模，调整学习率等超参数。  
更大的规模、调优超参数效果不佳：数据质量，收集更干净的数据  
训练集OK，测试集OK：完成。  
训练集OK，测试集差很多：收集数据、降低模型大小、改进正则化



在建立第一个端到端的系统后，就可以度量算法性能并决定如何改进算法。
许多机器学习新手都忍不住尝试很多不同的算法来进行改进。
然而，**收集更多的数据往往比改进学习算法要有用得多**。
<!-- % 414 mid -->


怎样判断是否要收集更多的数据？
首先，确定训练集上的性能是否可接受。
如果模型在训练集上的性能就很差，学习算法都不能在训练集上学习出良好的模型，那么就没必要收集更多的数据。  
反之，可以尝试增加更多的网络层或每层增加更多的隐藏单元，以增加模型的规模。
此外，也可以尝试调整学习率等超参数的措施来改进学习算法。  
> **[success]**  
> 场景：训练集上的性能很差（偏差大）  
> 是否需要收集更多的数据：否  
> 建议做法：增加模型的规模、调整超参数  

如果更大的模型和仔细调试的优化算法效果不佳，那么问题可能源自训练数据的\emph{质量}。
数据可能含太多噪声，或是可能不包含预测输出所需的正确输入。
这意味着我们需要重新开始，收集更干净的数据或是收集特征更丰富的数据集。  
> **[success]**  
> 场景：训练集上的性能很差、增加模型的规模和调整超参数无效  
> 是否需要收集更多的数据：是  
> 建议做法：收集更干净的数据或是收集特征更丰富的数据集    


如果训练集上的性能是可接受的，那么我们开始度量测试集上的性能。
如果测试集上的性能也是可以接受的，那么就顺利完成了。  
> **[success]**  
> 场景：训练集上的性能好、测试集上效果好  
> 是否需要收集更多的数据：否
> 建议做法：完成  

如果测试集上的性能比训练集的要差得多，那么收集更多的数据是最有效的解决方案之一。
这时主要的考虑是收集更多数据的代价和可行性，其他方法降低测试误差的代价和可行性，和增加数据数量能否显著提升测试集性能。
在拥有百万甚至上亿用户的大型网络公司，收集大型数据集是可行的，并且这样做的成本可能比其他方法要少很多，所以答案几乎总是收集更多的训练数据。
例如，收集大型标注数据集是解决对象识别问题的主要因素之一。  
> **[success]**  
> 场景：训练集上的性能好、测试集上效果差（方差大）、收集数据可行    
> 是否需要收集更多的数据：是   
> 建议做法：决定收集多少数据    

在其他情况下，如医疗应用，收集更多的数据可能代价很高或者不可行。
一个可以替代的简单方法是降低模型大小或是改进正则化（调整超参数，如权重衰减系数，或是加入正则化策略，如\,Dropout）。  
> **[success]**  
> 场景：训练集上的性能好、测试集上效果差、收集数据不可行    
> 是否需要收集更多的数据：否   
> 建议做法：降低模型规模、改进正则化    

如果调整正则化超参数后，训练集性能和测试集性能之间的差距还是不可接受，那么收集更多的数据是可取的。
  
# 决定收集多少数据

在决定是否收集更多的数据时，也需要确定收集多少数据。
如\fig?所示，绘制曲线显示训练集规模和泛化误差之间的关系是很有帮助的。
根据走势延伸曲线，可以预测还需要多少训练数据来达到一定的性能。
通常，加入总数目一小部分的样本不会对泛化误差产生显著的影响。
因此，建议在对数尺度上考虑训练集的大小，例如在后续的实验中倍增样本数目。


如果收集更多的数据是不可行的，那么改进泛化误差的唯一方法是改进学习算法本身。
这属于研究领域，并非对应用实践者的建议。