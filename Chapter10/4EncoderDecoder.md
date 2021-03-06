我们已经在\fig?看到RNN如何将输入序列映射成固定大小的向量，在\fig?中看到RNN如何将固定大小的向量映射成一个序列，在\fig?、\fig?、\fig?和\fig?中看到RNN如何将一个输入序列映射到等长的输出序列。

本节我们讨论如何训练RNN，使其将输入序列映射到不一定等长的输出序列。
这在许多场景中都有应用，如语音识别、机器翻译或问答，其中训练集的输入和输出序列的长度通常不相同（虽然它们的长度可能相关）。

我们经常将RNN的输入称为"上下文"。
我们希望产生此上下文的表示，$C$。
这个上下文$C$可能是一个概括输入序列$X=(x^{(1)},\cdots,x^{(n_x)})$的**向量或者向量序列**。

用于映射可变长度序列到另一可变长度序列最简单的RNN架构最初由{cho-al-emnlp14}提出，之后不久由{Sutskever-et-al-NIPS2014}独立开发，并且第一个使用这种方法获得翻译的最好结果。
前一系统是对另一个机器翻译系统产生的建议进行评分，而后者使用独立的循环网络生成翻译。
这些作者分别将该架构称为编码-解码或序列到序列架构，如\fig?所示。
这个想法非常简单：（1）**编码器**(encoder)或\,\textbf{**读取器**}\,(reader)或\,\textbf{输入}(input)RNN处理输入序列。
编码器输出上下文$C$（通常是最终隐藏状态的简单函数）。
(2)**解码器**(decoder)或\,\textbf{写入器}(writer)或\,\textbf{输出}(output)RNN则以固定长度的向量（如\fig?）为条件产生输出序列$Y=(y^{(1)}, \cdots, y^{(n_y)})$。
这种架构对比本章前几节提出的架构的创新之处在于**长度$n_x$和$n_y$可以彼此不同**，而之前的架构约束$n_x = n_y = \tau$。
在序列到序列的架构中，两个RNN**共同训练**以最大化$\log P( y^{(1)}, \cdots, y^{(n_y)} \mid x^{(1)},\cdots,x^{(n_x)} )$(关于训练集中所有$x$和$y$对的平均)。
编码器RNN的最后一个状态$h_{n_x}$通常被当作输入的表示$C$并作为解码器RNN的输入。

{% reveal %}
\begin{figure}[!htb]
\ifOpenSource
\centerline{\includegraphics{figure.pdf}}
\else
\centerline{\includegraphics{Chapter10/figures/rnn_encdec}}
\fi
\caption{在给定输入序列$(x^{(1)},x^{(2)},\cdots,x^{(n_x)})$的情况下学习生成输出序列$(y^{(1)},y^{(2)},\cdots,y^{(n_y)})$的编码器-解码器或序列到序列的RNN架构的示例。 
它由读取输入序列的编码器RNN以及生成输出序列（或计算给定输出序列的概率）的解码器RNN组成。
编码器RNN的最终隐藏状态用于计算一般为固定大小的上下文变量$C$，$C$表示输入序列的语义概要并且作为解码器RNN的输入。
}
\end{figure}
{% endreveal %}

如果上下文$C$是一个向量，则解码器RNN只是在\sec?描述的向量到序列RNN。
正如我们所见，向量到序列RNN至少有两种接受输入的方法。
输入可以被提供为RNN的初始状态，或连接到每个时间步中的隐藏单元。
这两种方式也可以结合。

这里并不强制要求编码器与解码器的隐藏层具有相同的大小。

此架构的一个明显不足是，编码器RNN输出的上下文$C$的维度太小而难以适当地概括一个长序列。
这种现象由{Bahdanau-et-al-ICLR2015-small}在机器翻译中观察到。
他们提出让$C$成为可变长度的序列，而不是一个固定大小的向量。
此外，他们还引入了将序列$C$的元素和输出序列的元素相关联的**注意力机制**（attention mechanism）。
读者可在\sec?了解更多细节。
