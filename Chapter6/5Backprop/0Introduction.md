前向传播：$x \rightarrow$ 隐藏层 $\rightarrow \hat y \rightarrow J(\theta)$  
反向传播：$$J(\theta)$$的信息通过网络向后（输入层）流动。  

*注意：输入层是后，输出层是前。*  

反向传播算法是一种计算梯度的算法。  
但它不局限于计算代价函数关于参数的梯度。  