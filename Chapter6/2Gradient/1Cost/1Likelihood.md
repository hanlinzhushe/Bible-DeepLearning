大多数神经网络使用[最大似然估计](https://windmising.gitbook.io/mathematics-basic-for-ml/gai-shuai-lun/likelihood)来训练，由此推导出来的代价函数为：  
$$
J(\theta) = -E_{X,y \sim \hat P_{data}}\log p_{model}(y|x)  \tag{1}
$$

使用这种代价函数的优点是：  
1. 不用为每个模型设计代价函数，p(y|x)定了代价函数就这了。  
2. log的形式可以消除某些单元的指数效果，避免神经元饱和。  
3. [?]应用于实践中时通常没有最小值
