 






# 将噪声作用于输入 

> **[success]**  
这种方法等价于“对权重施加范数惩罚”。  

\secref{sec:dataset_augmentation_chap7}已经提出将噪声作用于输入，作为数据集增强策略。
对于某些模型而言，向输入添加方差极小的噪声等价于对权重施加范数惩罚\citep{Bishop1995,bishop95training}。  

# 将噪声作用于隐藏单元  

> **[success]**  
效果强大。例如[dropout](https://windmissing.github.io/Bible-DeepLearning/Chapter7/12Dropout.html)  

在一般情况下，注入噪声远比简单地收缩参数强大，特别是噪声被添加到隐藏单元时会更加强大。
向隐藏单元添加噪声是值得单独讨论重要的话题；在\secref{sec:dropout}所述Dropout算法是这种做法的主要发展方向。

# 将噪声作用于权重  

> **[success]**  
主要用于RNN。  
因为它推动模型进入对权重小的变化相对不敏感的区域，找到的点不只是极小点  

另一种正则化模型的噪声使用方式是将其加到权重。
这项技术主要用于循环神经网络\citep{JimGilesHorne1996,Graves-2011}。
这可以被解释为关于权重的贝叶斯推断的随机实现。  
> **[warning]** [?] 关于权重的贝叶斯推断的随机实现  

贝叶斯学习过程将权重视为不确定的，并且可以通过概率分布表示这种不确定性。  
> **[success]**  
权重不是一个值，而是关于所有可能的值的分布。  

向权重添加噪声是反映这种不确定性的一种实用的随机方法。

在某些假设下，施加于权重的噪声可以被解释为与更传统的正则化形式等同，鼓励要学习的函数保持稳定。
我们研究回归的情形，也就是训练将一组特征$x$映射成一个标量的函数$\hat y(x)$，并使用最小二乘代价函数衡量模型预测值$\hat y(x)$与真实值$y$的误差：  
$$
\begin{aligned}
 J = \Bbb E_{p(x,y)}[(\hat y(x) - y)^2].
\end{aligned}
$$

训练集包含$m$对标注样例$\{(x^{(1)}, y^{(1)}),\cdots,(x^{(m)}, y^{(m)})\}$。

现在我们假设对每个输入表示，网络权重添加随机扰动$\epsilon_{w} \sim \Bbb N(\epsilon;0, \eta I \, )$。
想象我们有一个标准的$l$层MLP。
我们将扰动模型记为$\hat y_{\epsilon_{W}} (x)$。
尽管有噪声注入，我们仍然希望减少网络输出误差的平方。
因此目标函数变为：  
$$
\begin{aligned}
 \tilde J_{W} &= \Bbb E_{p(x,y,\epsilon_{W})}[(\hat y_{\epsilon_{W}}(x) - y)^2] \\
   &=  \Bbb E_{p(x,y,\epsilon_{W})}[\hat y_{\epsilon_{W}}^2(x) -  2y\hat y_{\epsilon_{W}}
   (x)+ y^2] .
\end{aligned}
$$

> **[warning]** 上面公式中，${\epsilon_{W}}$体现在哪？  

对于小的$\eta$，最小化带权重噪声（方差为$\eta I$\,）的$J$等同于最小化附加正则化项：
$ \eta \Bbb E_{p(x,y)}[||\nabla_{W}\hat y(x)||^2]$的$J$。
这种形式的正则化鼓励参数进入权重小扰动对输出相对影响较小的参数空间区域。
换句话说，它**推动模型进入对权重小的变化相对不敏感的区域**，找到的点不只是极小点，还是由平坦区域所包围的极小点\citep{Hochreiter95}。
在简化的线性回归中（例如，$\hat y(x) = w^\top x + b$），正则项退化为$ \eta \Bbb E_{p(x)}[||x||^2]$，这与函数的参数无关，因此不会对$\tilde J_{w}$关于模型参数的梯度有影响。

# 将噪声作用于输出  

> **[success]**  
这种方法假设标签y有一定的错误。  
因此显式地对标签上的噪声进行建模。  

大多数数据集的$y$标签都有一定错误。
错误的$y$不利于最大化$\log p(y \mid x)$。
避免这种情况的一种方法是显式地对标签上的噪声进行建模。
例如，我们可以假设，对于一些小常数$\epsilon$，训练集标记$y$是正确的概率是$1-\epsilon$，（以$\epsilon$的概率）任何其他可能的标签也可能是正确的。  
> **[success]**  
假设y有错误，且错误率是$\epsilon$  

这个假设很容易就能解析地与代价函数结合，而不用显式地抽取噪声样本。
例如，**标签平滑**（label smoothing）通过把确切分类目标从0和1替换成$\frac{\epsilon}{k-1}$和$1-\epsilon$，正则化具有$k$个输出的softmax函数的模型。  
> **[success]**  
原目标：0和1  
新目标：$\frac{\epsilon}{k-1}$和$1-\epsilon$  
原softmax：无正则化的softmax  
新softmax：正则化的softmax  
代价函数：交叉熵代价函数  
[?] 怎么正则化softmax?  

标准交叉熵损失可以用在这些非确切目标的输出上。
使用softmax函数和明确目标的最大似然学习可能永远不会收敛——
softmax函数永远无法真正预测0概率或1概率，因此它会继续学习越来越大的权重，使预测更极端。  
> **[success]**  
原目标+原softmax = 权重越来越大，预测更极端  

使用如权重衰减等其他正则化策略能够防止这种情况。
标签平滑的优势是能够防止模型追求确切概率而不影响模型学习正确分类。  
> **[success]**  
新目标+新softmax = 防止模型追求确切概率而不影响模型学习正确分类  

这种策略自20世纪80年代就已经被使用，并在现代神经网络继续保持显著特色\citep{Szegedy-et-al-2015}。
