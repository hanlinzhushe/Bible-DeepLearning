关于隐藏单元的设计，目标没有多少明确的指导性原则。  
一些隐藏单元不是在所有的输入上都可微，但仍表现不错。  
下文默认隐藏单元会先对向量x做仿射变换$$z=W^Tx+b$$，然后应用逐元素的非线性函数g(z)  

*隐藏单元=中间层，不同材料使用的术语不同。*