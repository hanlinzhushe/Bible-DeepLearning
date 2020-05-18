当一个机器学习系统效果不好时，通常很难判断效果不好的原因是算法本身，还是算法实现错误。
由于各种原因，机器学习系统很难调试。


<!-- % 424 -->
在大多数情况下，我们不能提前知道算法的行为。
事实上，使用机器学习的整个出发点是，它会发现一些我们自己无法发现的有用行为。
如果我们在一个\emph{新}的分类任务上训练一个神经网络，它达到$5\%$的测试误差，我们没法直接知道这是期望的结果，还是次优的结果。


另一个难点是，大部分机器学习模型有多个自适应的部分。
如果一个部分失效了，其他部分仍然可以自适应，并获得大致可接受的性能。
例如，假设我们正在训练多层神经网络，其中参数为权重$W$和偏置 $b$。
进一步假设，我们单独手动实现了每个参数的梯度下降规则。
而我们在偏置更新时犯了一个错误：
$$
\begin{aligned}
	b \leftarrow b - \alpha,
\end{aligned}
$$

其中$\alpha$是学习率。
这个错误更新没有使用梯度。
它会导致偏置在整个学习中不断变为负值，对于一个学习算法来说这显然是错误的。 
然而只是检查模型输出的话，该错误可能并不是显而易见的。
根据输入的分布，权重可能可以自适应地补偿负的偏置。
<!-- % -- 425 head -->


大部分神经网络的调试策略都是解决这两个难题的一个或两个。
我们可以设计一种足够简单的情况，能够提前得到正确结果，判断模型预测是否与之相符；我们也可以设计一个测试，独立检查神经网络实现的各个部分。


一些重要的调试检测如下所列。

\emph{可视化计算中模型的行为}：%??  in action 
当训练模型检测图像中的对象时，查看一些模型检测到部分重叠的图像。
在训练语音生成模型时，试听一些生成的语音样本。
这似乎是显而易见的，但在实际中很容易只注意量化性能度量，如准确率或对数似然。
直接观察机器学习模型运行其任务，有助于确定其达到的量化性能数据是否看上去合理。
错误评估模型性能可能是最具破坏性的错误之一，因为它们会使你在系统出问题时误以为系统运行良好。
<!-- % 425 mid -->


\emph{可视化最严重的错误}：
大多数模型能够输出运行任务时的某种置信度量。
例如，基于\,softmax函数\,输出层的分类器给每个类分配一个概率。
因此，分配给最有可能的类的概率给出了模型在其分类决定上的置信估计值。
通常，相比于正确预测的概率最大似然训练会略有高估。
但是由于实际上模型的较小概率指示结果对应着正确的标签的可能也较小，因此它们在一定意义上还是有些用的。
通过查看训练集中很难正确建模的样本，通常可以发现该数据预处理或者标记方式的问题。
例如，街景转录系统原本有个问题是，地址号码检测系统会将图像裁剪得过于紧密，而省略掉了一些数字。
然后转录网络会给这些图像的正确答案分配非常低的概率。
将图像排序，确定置信度最高的错误，显示系统的裁剪有问题。
修改检测系统裁剪更宽的图像，使整个系统获得了更好的性能，即便转录网络需要能够处理地址号码中位置和范围更大变化的情况。
<!-- % 425 end -->


\emph{根据训练和测试误差检测软件}：
我们往往很难确定底层软件是否是正确实现。
训练和测试误差能够提供一些线索。
如果训练误差较低，但是测试误差较高，那么很有可能训练过程是在正常运行，但模型由于算法原因过拟合了。
另一种可能是，测试误差没有被正确地度量，可能是由于训练后保存模型再重载去度量测试集时出现问题，或者是因为测试数据和训练数据预处理的方式不同。
如果训练和测试误差都很高，那么很难确定是软件错误，还是由于算法原因模型欠拟合。
这种情况需要进一步的测试，如下面所述。

<!-- % -- 426 head -->

\emph{拟合极小的数据集}：
当训练集上有很大的误差时，我们需要确定问题是真正的欠拟合，还是软件错误。
通常，即使是小模型也可以保证很好地拟合一个足够小的数据集。
例如，只有一个样本的分类数据可以通过正确设置输出层的偏置来拟合。
通常，如果不能训练一个分类器来正确标注一个单独的样本，或不能训练一个自编码器来成功地精准再现一个单独的样本，或不能训练一个生成模型来一致地生成一个单独的样本，那么很有可能是由于软件错误阻止训练集上的成功优化。
此测试可以扩展到只有少量样本的小数据集上。

<!-- % 426 mid -->

\emph{比较反向传播导数和数值导数}：
如果读者正在使用一个需要实现梯度计算的软件框架，或者在添加一个新操作到求导库中，必须定义它的\texttt{bprop}方法，那么常见的错误原因是没能正确地实现梯度表达。
验证这些求导正确性的一种方法是比较实现的自动求导和通过**有限差分(finite difference)**计算的导数。
因为
$$
\begin{aligned}
	f'(x) = \lim_{\epsilon \to 0} \frac{f(x+\epsilon) - f(x)}{\epsilon},
\end{aligned}
$$

我们可以使用小的、有限的$\epsilon$近似导数：
$$
\begin{aligned}
	f'(x) \approx \frac{f(x+\epsilon) - f(x)}{\epsilon}.
\end{aligned}
$$

我们可以使用**中心差分(centered difference)**提高近似的准确率：
$$
\begin{aligned}
	f'(x) \approx \frac{ f(x+\frac{1}{2}\epsilon) - f(x-\frac{1}{2}\epsilon) }{\epsilon}.
\end{aligned}
$$

扰动大小$\epsilon$必须足够大，以确保该扰动不会由于数值计算的有限精度问题产生舍入误差。
> **[success]** from Ag  
> 有限差分 = 单边误差  
> 中心差分 = 双边误差  
> 双边误差比单元误差更准确  
> **基于双边误差计算参数$\theta$的导数**  
> (1)把所有参数W、b合成一个大的向量$\theta$  
> (2)所有参数的导数dW、db也看成是一个大的向量$d\theta$  
> (3)$d\theta[i]_{appro} \approx \frac{J(\theta_1, \theta_2, \cdots, \theta_i+\epsilon,\cdots) - J(\theta_1, \theta_2, \cdots, \theta_i-\epsilon,\cdots)}{2\epsilon}$  
> (4)比较$d\theta[i]_{appro}$和$d\theta[i]$，比较方法为计算以下公式与一个threshold    
$$
\frac{||d\theta[i]_{appro}-d\theta[i]||_2}{||\theta[i]_{appro}||_2 + ||\theta[i]||_2}
$$

> **使用双边误差计算导数的注意事项**
> 这种计算$d\theta$的方式很慢，仅用于debug，不用于训练  
> 如果check过高，先检查是哪个参数（W还是b，哪个index）引起的。  
> 计算时不要忘记正则化项  
> debug时不要开启dropout  

通常，我们会测试向量值函数$g:\Bbb R^m \to \Bbb R^n$的梯度或\,Jacobian\,矩阵。
令人遗憾的是，有限差分只允许我们每次计算一个导数。
我们可以使用有限差分 $mn$次评估$g$的所有偏导数，也可以将该测试应用于一个新函数（在函数$g$的输入输出都加上随机投影）。%??  后面这句好像不对？  g的输入输出都使用随机投影的  
> **[warning]** 在函数$g$的输入输出都加上随机投影是什么意思？  

例如，我们可以将导数实现的测试用于函数$f(x) = u^T g(v x)$，其中$u$和$v$是随机向量。%??  这句也不对
正确计算$f'(x)$要求能够正确地通过$g$反向传播，但是使用有限差分能够高效地计算，因为$f$只有一个输入和一个输出。
通常，一个好的方法是在多个$u$值和$v$值上重复这个测试，可以减少测试忽略了垂直于随机投影的错误的几率。%??  很难  
> **[warning]** 看不懂？  

<!-- % 427 head -->

如果我们可以在复数上进行数值计算，那么使用复数作为函数的输入会有非常高效的数值方法估算梯度{cite?}。
该方法基于如下观察
\begin{aligned}
	f(x + i\epsilon) &= f(x) + i\epsilon f'(x) + O(\epsilon^2) ,\\
	\text{real}( f(x+i\epsilon) ) &= f(x) + O(\epsilon^2), \quad \text{image}( \frac{f(x+i\epsilon)}{ \epsilon } ) = f'(x) + O(\epsilon^2),
\end{aligned}
其中$i=\sqrt{-1}$。
和上面的实值情况不同，这里不存在消除影响，因为我们对$f$在不同点上计算差分。
因此我们可以使用很小的$\epsilon$，比如$\epsilon = 10^{-150}$，其中误差$O(\epsilon^2)$对所有实用目标都是微不足道的。

<!-- % 427 mid -->

\emph{监控激活函数值和梯度的直方图}：
可视化神经网络在大量训练迭代后（也许是一个轮）收集到的激活函数值和梯度的统计量往往是有用的。
隐藏单元的预激活值可以告诉我们该单元是否饱和，或者它们饱和的频率如何。
例如，对于整流器，它们多久关一次？是否有单元一直关闭？
对于双曲正切单元而言，预激活绝对值的平均值可以告诉我们该单元的饱和程度。
在深度网络中，传播梯度的快速增长或快速消失，可能会阻碍优化过程。
最后，比较参数梯度和参数的量级也是有帮助的。
正如{cite?}所建议的，我们希望参数在一个小批量更新中变化的幅度是参数量值$1\%$这样的级别，而不是$50\%$或者$0.001\%$（这会导致参数移动得太慢）。
也有可能是某些参数以良好的步长移动，而另一些停滞。
如果数据是稀疏的（比如自然语言），有些参数可能很少更新，检测它们变化时应该记住这一点。

<!-- % 428 head -->

最后，许多深度学习算法为每一步产生的结果提供了某种保证。
例如，在第三部分，我们将看到一些使用代数解决优化问题的近似推断算法。
通常，这些可以通过测试它们的每个保证来调试。
某些优化算法提供的保证包括，目标函数值在算法的迭代步中不会增加，某些变量的导数在算法的每一步中都是零，所有变量的梯度在收敛时会变为零。
通常，由于舍入误差，这些条件不会在数字计算机上完全成立，因此调试测试应该包含一些容差参数。