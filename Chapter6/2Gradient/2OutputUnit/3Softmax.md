# 什么是Softmax Unit?

任何时候当我们想要表示一个具有n个可能取值的离散型随机变量的分布时，我们都可以使用softmax函数。
它可以看作是sigmoid函数的扩展，其中sigmoid函数用来表示二值型变量的分布。

> **[success]**  
Linear Unit用于回归问题根据x预测y的均值。  
Sigmoid Unit用于二分类问题根据x预测y=1的概率。  
Softmax Unit用于多分类问题根据x预测y=a的概率。  

softmax函数最常用作分类器的输出，来表示n个不同类上的概率分布。
比较少见的是，softmax函数可以在模型内部使用，例如如果我们想要在某个内部变量的$n$个不同选项中进行选择。

在二值型变量的情况下，我们希望计算一个单独的数

$$
\hat{y} = P(y=1\mid x)
$$

因为这个数需要处在0和1之间，并且我们想要让这个数的对数可以很好地用于对数似然的基于梯度的优化，我们选择去预测另外一个数$$z=\log \hat{P}(y=1\mid x)$$。  

> **[success]**  
利用$$\log \hat P(y=1\mid x)$$和$$\hat{P}(y=1\mid x)$$的等价性。  
**这是一个很好的数学思路。**
这一点在这本书中广泛使用。  

对其指数化和归一化，我们就得到了一个由sigmoid函数控制的Bernoulli分布。

为了推广到具有n个值的离散型变量的情况，我们现在需要创造一个向量$$\hat{y}$$，它的每个元素是$$\hat{y}_i = P(y=i\mid x)$$。
我们不仅要求每个$$\hat{y}_i$$元素介于0和1之间，还要使得整个向量的和为1，使得它表示一个有效的概率分布。
用于Bernoulli分布的方法同样可以推广到Multinoulli分布。  
> **[success]** [Multinoulli分布](TODO)  

首先，线性层预测了未归一化的对数概率：  

$$
z = W^\top h+b
$$

其中$$z_i=\log \hat{P}(y=i\mid x)$$。
softmax函数然后可以对z指数化和归一化来获得需要的$$\hat{y}$$。
最终，softmax函数的形式为  

$$
\text{softmax}(z)_i = \frac{\exp(z_i)}{\sum_j \exp(z_j)}
$$

> **[success]** 和sigmoid和推导过程类似  
（1）假设$$z_i$$与$$\log \hat{P}(y=i\mid x)$$呈线性关系，即$$z_i=\log \hat{P}(y=i\mid x)$$  
（2）计算$$\hat{P}(y=i\mid x) = \exp z_i$$  
（3）将$$\hat{P}(y=i\mid x)$$归一化，即$${P}(y=i\mid x) = \frac{\exp(z_i)}{\sum_j \exp(z_j)}$$  
（4）代入cross entropy代价函数的公式：$$J(\theta) = E[\log {P}(y=i\mid x)] = E[z_i - \log \sum_j \exp(z_j)]$$  

# Softmax Unit的最大似然代价函数

和logistic sigmoid一样，当使用最大化对数似然训练softmax来输出目标值$$y$$时，使用指数函数工作地非常好。
这种情况下，我们想要最大化$$\log P(y =i; z)=\log \text{softmax}(z)_i$$。  
> **[warning]** [?] 为什么要最大化$$\log P(y =i; z)$$？此处假设y=i是正确答案？  

将softmax定义成指数的形式是很自然的因为对数似然中的log可以抵消softmax中的exp：  

$$
\log \text{softmax}(z)_i = z_i - \log \sum_j \exp(z_j)
$$
公式6.30  

公式6.30中的第一项表示输入$$z_i$$总是对代价函数有直接的贡献。
因为这一项不会饱和，所以即使$$z_i$$对公式6.30的第二项的贡献很小，学习依然可以进行。
当最大化对数似然时，第一项鼓励$$z_i$$被推高，而第二项则鼓励所有的z被压低。
为了对第二项$$\log \sum_j \exp(z_j)$$有一个直观的理解，注意到这一项可以大致近似为$$\max_j z_j$$。
这种近似是基于对任何明显小于$$\max_j z_j$$的$$z_k$$，$$\exp(z_k)$$都是不重要的。  
> **[warning]** [?] [?] 这一段看不懂   

我们能从这种近似中得到的直觉是，负对数似然代价函数总是**强烈地惩罚最活跃的不正确预测**。
如果正确答案已经具有了softmax的最大输入，那么$$-z_i$$项和$$\log\sum_j \exp(z_j) \approx \max_j z_j = z_i$$项将大致抵消。
这个样本对于整体训练代价贡献很小，这个代价主要由其他未被正确分类的样本产生。

到目前为止我们只讨论了一个例子。
总体来说，未正则化的最大似然会驱动模型去学习一些参数，而这些参数会驱动softmax函数来预测在训练集中观察到的每个结果的比率：  

$$
\text{softmax}(z(x; \theta))_i \approx \frac{\sum_{j=1}^m {1}_{y^{(j)}=i, x^{(j)} = x}  }{ \sum_{j=1}^{m} {1}_{x^{(j)} = x} }
$$

因为最大似然是一致的估计量，所以只要模型族能够表示训练的分布，这就能保证发生。
在实践中，有限的模型能力和不完美的优化将意味着模型只能近似这些比率。
> **[success]**  
有限的模型能力: 模型族不能够表示训练的分布  
不完美的优化：需要引入正则化或其它优化方法  

除了对数似然之外的许多目标函数对softmax函数不起作用。
具体来说，那些不使用对数来抵消softmax中的指数的目标函数，当指数函数的变量取非常小的负值时会造成梯度消失，从而无法学习。  
> **[success]**   
softmax Unit在某些情况下会饱和，下面会具体介绍。  
因此要设计能抵消饱和（或对其进行补偿）的代价函数。  

特别是，平方误差对于softmax单元来说是一个很差的损失函数，即使模型做出高度可信的不正确预测，也不能训练模型改变其输出。  
> **[warning]** 也许MSE效果不好，但为什么说它没有作用呢？  

# 为什么Softmax Unit会饱和？

要理解为什么这些损失函数可能失败，我们需要检查softmax函数本身。

像sigmoid一样，softmax激活函数可能会饱和。
sigmoid函数具有单个输出，当它的输入极端负或者极端正时会饱和。
对于softmax的情况，它有多个输出值。
当输入值之间的差异变得极端时，这些输出值可能饱和。
当softmax饱和时，基于softmax的许多代价函数也饱和，除非它们能够转化饱和的激活函数。

为了说明softmax函数对于输入之间差异的响应，观察到当对所有的输入都加上一个相同常数时softmax的输出不变：  

$$
\text{softmax}(z) = \text{softmax}(z+c)
$$

使用这个性质，我们可以导出一个数值方法稳定的softmax函数的变体：  

$$
\text{softmax}(z) = \text{softmax}(z- \max_i z_i)
$$

变换后的形式允许我们在对softmax函数求值时只有很小的数值误差，即使是当z包含极正或者极负的数时。
观察softmax数值稳定的变体，可以看到softmax函数由它的变量偏离$$\max_i z_i$$的量来驱动。

当其中一个输入是最大（$$z_i = \max_i z_i$$）并且$$z_i$$远大于其他的输入时，相应的输出$$\text{softmax}(z)_i$$会饱和到1。
当$$z_i$$不是最大值并且最大值非常大时，相应的输出$$\text{softmax}(z)_i$$也会饱和到0。
这是sigmoid单元饱和方式的一般化，并且如果损失函数不被设计成对其进行补偿，那么也会造成类似的学习困难。  
> **[success]**  
当z中的最大值与其它值差距很大时，$$softmax(z_{max})$$趋于1，而$$softmax(z_{other})$$趋于0，此时饱和。  
若最大的z正好代表正确答案，那么是正常的。   
若最大的z不是代表正确答案，那么是不正确的饱和，会导致学习缓慢。  

# 变量z的产生

softmax函数的变量z可以通过两种方式产生。
最常见的是简单地使神经网络中softmax之前的层输出z的每个元素，就像先前描述的使用线性层$$z={W}^\top h+b$$。
虽然很直观，但这种方法是对分布的过度参数化。
n个输出总和必须为1的约束意味着只有n-1个参数是必要的；第n个概率值可以通过1减去前面n-1个概率来获得。
因此，我们可以强制要求z的一个元素是固定的。
例如，我们可以要求$$z_n=0$$。  
> **[warning]** [?]  [?] 这一段看不懂   

事实上，这正是sigmoid单元所做的。
定义$$P(y=1\mid x)=\sigma(z)$$等价于用二维的z以及$$z_1=0$$来定义$$P(y=1\mid x)=\text{softmax}(z)_1$$。  
> **[warning]** [?]  [?] 这一段看不懂   

无论是n-1个变量还是n个变量的方法，都描述了相同的概率分布，但会产生不同的学习机制。
在实践中，无论是过度参数化的版本还是限制的版本都很少有差别，并且实现过度参数化的版本更为简单。

# 其它

从神经科学的角度看，有趣的是认为softmax是一种在参与其中的单元之间形成竞争的方式：softmax输出总是和为1，所以一个单元的值增加必然对应着其他单元值的减少。
这与被认为存在于皮质中相邻神经元间的侧抑制类似。
在极端情况下（当最大的$$a_i$$和其他的在幅度上差异很大时），它变成了赢者通吃的形式（其中一个输出接近1，其他的接近0）。

“softmax”的名称可能会让人产生困惑。
这个函数更接近于argmax函数而不是max函数。  
> **[success]** [argmax](https://baike.baidu.com/item/argmax/6034072?fr=aladdin)  

“soft”这个术语来源于softmax函数是连续可微的。
“argmax”函数的结果表示为一个one-hot向量（只有一个元素为1，其余元素都为0的向量），不是连续和可微的。
softmax函数因此提供了argmax的“软化”版本。max函数相应的软化版本是$$\text{softmax}(z)^\top z$$。  
> **[warning]** soft版本是怎么计算的？  

可能最好是把softmax函数称为“softargmax”，但当前名称已经是一个根深蒂固的习惯了。