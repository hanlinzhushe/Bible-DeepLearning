如\secref{sec:manifold_learning}所述，许多机器学习通过假设数据位于低维流形附近来克服维数灾难。  
> **[warning]**  低维流形？  维数灾难？  

# 切面距离算法  

一个利用流形假设的早期尝试是**切面距离**（tangent distance）算法\citep{Simard93-small,Simard98}。
它是一种非参数的最近邻算法，其中使用的度量不是通用的欧几里德距离，而是根据邻近流形关于聚集概率的知识导出的。  
> **[warning]** 流形：局部具有欧几里德空间性质的空间。  [?] 邻近流形聚集概率  

这个算法**假设我们尝试分类的样本和同一流形上的样本具有相同的类别**。  
> **[success]** 流形假设  

由于分类器应该对局部因素（对应于流形上的移动）的变化保持不变，一种合理的度量是将点$x_1$和$x_2$各自所在流形$M_1$和$M_2$的距离作为点$x_1$和$x_2$之间的最近邻距离。  
> **[warning]** 流形上的移动?  

然而这可能在计算上是困难的（它需要解决一个寻找$M_1$和$M_2$最近点对的优化问题），一种局部合理的廉价替代是使用$x_i$点处切平面近似$M_i$，并测量两条切平面或一个切平面和点之间的距离。  
> **[warning]** x点处切平面近似M？  

这可以通过求解一个低维线性系统（就流形的维数而言）来实现。  
> **[warning]** 低维线性系统？切向量？  

当然，这种算法需要指定那些切向量。  
> **[success]**  
类似于最近邻算法。  
距离度量：欧氏距离 -> 邻近流行聚焦概率  
邻近流行聚焦概率计算：x1、x2各自所在的流形M1、M2r距离 -> x1、x2的切面距离  

# 正切传播算法  

 

受相关启发，**正切传播**（tangent prop）算法\citep{Simard92-short}（\figref{fig:chap7_mtc_color}）训练带有额外惩罚的神经网络分类器，**使神经网络的每个输出$f(x)$对已知的变化因素是局部不变的**。  
> **[success]** 代价函数的目标  

这些**变化因素对应于沿着的相同样本聚集的流形的移动**。
这里实现局部不变性的方法是**要求$\nabla_{x} f(x)$与已知流形的切向$v^{(i)}$正交**，或者等价地**通过正则化惩罚$\Omega$使$f$在$x$的$v^{(i)}$方向的导数较小**：    
> **[success]** 达到目标的方法：设置正则化项  

$$
\begin{aligned}
 \Omega(f) = \sum_i \Big((\nabla_{x} f(x)^\top v^{(i)}) \Big)^2 .
\end{aligned}
$$

这个正则化项当然可以通过适当的超参数缩放，并且对于大多数神经网络，我们需要对许多输出求和(此处为描述简单，$f(x)$为唯一输出)。
与切面距离算法一样，切向量是从先验知识推导的，通常是从变换（如平移、旋转和缩放图像）的效果获得的形式知识。  
> **[warning]**  推导先验？形式知识？  

正切传播不仅用于监督学习\citep{Simard92-short}，还在强化学习\citep{Thrun-NIPS1994}中有所应用。  

{% reveal %}
{% raw %}
\begin{figure}[!htb]
\ifOpenSource
\centerline{\includegraphics{figure.pdf}}
\else
\centerline{\includegraphics{Chapter7/figures/mtc_color}}
\fi
\caption{正切传播算法\citep{Simard92-short}和流形正切分类器主要思想的示意图\citep{Dauphin-et-al-NIPS2011-small}，它们都正则化分类器的输出函数$f(x)$。
每条曲线表示不同类别的流形，这里表示嵌入二维空间中的一维流形。
在一条曲线上，我们选择单个点并绘制一个与类别流形相切（平行并接触流形）的向量以及与类别流形垂直（与流形正交）的向量。
在多维情况下，可以存在许多切线方向和法线方向。
我们希望分类函数在垂直于流形方向上快速改变，并且在类别流形的方向上保持不变。
正切传播和流形正切分类器都会正则化 $f(x)$，使其不随$x$沿流形的移动而剧烈变化。
正切传播需要用户手动指定正切方向的计算函数（例如指定小平移后的图像保留在相同类别的流形中），而流形正切分类器通过训练自编码器拟合训练数据来估计流形的正切方向 。
我们将在\chapref{chap:autoencoders}中讨论使用自编码器来估计流形。
}
\label{fig:chap7_mtc_color}
\end{figure}
{% endraw %}
{% endreveal %}

## 正切传播算法 + 数据集增加

正切传播与数据集增强密切相关。
在这两种情况下，该算法的用户通过指定一组应当不会改变网络输出的转换，将其先验知识编码至算法中。  
> **[success]**  
先验知识是指：将图像平移、旋转、缩放后不影响图像的分类结果。  

不同的是在数据集增强的情况下，网络显式地训练正确分类这些施加大量变换后产生的不同输入。  
> **[success]**  
普通方法：  
原始图像 ---> 训练   
原始图像 ---> 使用先验知识生成的图像 ---> 训练  

正切传播不需要显式访问一个新的输入点。
取而代之，它解析地对模型正则化从而在指定转换的方向抵抗扰动。
> **[success]**  
高级方法：  
原始图像 ---> 训练   
原始图像 + 先验知识 ---> 训练 

虽然这种解析方法是聪明优雅的，但是它有两个主要的缺点。  
> **[warning]** 缺点没看懂  

首先，模型的正则化只能抵抗无穷小的扰动。
显式的数据集增强能抵抗较大的扰动。
其次，我们很难在基于整流线性单元的模型上使用无限小的方法。
这些模型只能通过关闭单元或缩小它们的权重才能缩小它们的导数。
它们不能像\ENNAME{sigmoid}或\ENNAME{tanh}单元一样通过较大权重在高值处饱和以收缩导数。
数据集增强在整流线性单元上工作得很好，因为不同的整流单元会在每一个原始输入的不同转换版本上被激活。

正切传播也和双反向传播\citep{DruckerLeCun92}以及对抗训练\citep{Szegedy-ICLR2014,Goodfellow-2015-adversarial}有关联。
双反向传播正则化使Jacobian矩阵偏小，而对抗训练找到原输入附近的点，训练模型在这些点上产生与原来输入相同的输出。
正切传播和手动指定转换的数据集增强都要求模型在输入变化的某些特定的方向上保持不变。
双反向传播和对抗训练都要求模型对输入所有方向中的变化（只要该变化较小）都应当保持不变。
正如数据集增强是正切传播非无限小的版本，对抗训练是双反向传播非无限小的版本。
> **[warning]** 这一段不懂    

# 流形正切分类器

流形正切分类器\citep{Dauphin-et-al-NIPS2011}无需知道切线向量的先验。  
> **[success]**  
前面两种算法需要计算一些先验知道。  
流形正切分类器先通过无监督学习算法把这些先验知识算出来。  

我们将在\chapref{chap:autoencoders}看到，自编码器可以估算流形的切向量。
流形正切分类器使用这种技术来避免用户指定切向量。
如\figref{fig:chap14_cifar_cae}所示，这些估计的切向量不仅对图像经典几何变换（如转化、旋转和缩放）保持不变，还必须掌握对特定对象（如正在移动的身体某些部分）保持不变的因素。
因此根据流形正切分类器提出的算法相当简单：
（1）使用自编码器通过无监督学习来学习流形的结构，以及（2）如正切传播（\eqnref{eq:767}）一样使用这些切面正则化神经网络分类器。

在本章中，我们已经描述了大多数用于正则化神经网络的通用策略。
正则化是机器学习的中心主题，因此我们将不时在其余各章中重新回顾。
机器学习的另一个中心主题是优化，我们将在下一章描述。