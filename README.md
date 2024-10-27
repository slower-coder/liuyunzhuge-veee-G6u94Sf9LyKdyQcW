
全面解释人工智能LLM模型的真实工作原理（一）


# 人工智能 \#大语言模型LLM \#机器学习ML \#深度学习 \#数据挖掘


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241026195657778-1629671028.png)


序言：为了帮助更多人理解，我们将分成若干小节来讲解大型语言模型（LLM）的真实工作原理，从零开始，不需额外知识储备，只需初中数学基础（懂加法和乘法就行）。本文包含理解 LLM 所需的全部知识和概念，是完全自包含的（不依赖外部资料）。我们首先将在纸上构建一个简单的生成式大语言模型，然后逐步剖析每一步细节，帮助你掌握现代人工智能语言模型（LLM）和 Transformer 架构。文中去掉了所有复杂术语和机器学习专业名词，简化为纯粹的数字乘法与加法表达。当然我们并没有舍弃细节，会在文中适当位置指出相关术语，以便你能与其他专业内容建立关联。


从懂得加法/乘法数学运算到搞清当今最先进的AI模型，意味着我们需要覆盖大量内容。这不是一个玩具版的LLM解释——有心人理论上可以从中重建一个现代LLM。我删去了一切多余的字句，因此整篇知识并不适合快速浏览式阅读。


**（关注并订阅作者，您将能及时收到作者的最新更新、行业内最新技术动态以及实际动手实践的经验分享。）**


本文全部内容将涵盖下列15个要点：


1. 一个简单的神经网络
2. 这些神经网络模型是如何训练的？
3. 模型是如何生成输出语言的？
4. 是什么让LLM模型效果如此出色？
5. 嵌入(Embedding)
6. 子词分词器
7. 自注意力
8. Softmax
9. 残差连接
10. 层归一化
11. Dropout
12. 多头注意力
13. 位置嵌入
14. GPT架构
15. Transformer架构


我们开始吧。


首先要提到的是，神经网络只能接收数字作为输入，并输出数字。毫无例外，魔法的关键在于如何将用户的一切输入内容（文字、图像、视频、声音）转换为数字，以及对神经网络的输出数字进行解释以达到目的。最后，我们自己构建一个神经网络，使其接收你提供的输入并给出你想要的输出（基于你选择的输出解码方式）。让我们来看看如何从加法和乘法这些基本运算原理出发，实现人工智能语言模型 Llama 3\.1 能力的效果。


构建一个简单的神经网络：


我们先来搞清楚一个可以对物体进行分类的简单神经网络是怎样的存在，这是讲述我们要设计的神经网络的任务背景：


• 用颜色（RGB）和体积（毫升）来描述将被神经网络识别的物体


• 要求神经网络准确分辨出物体到底是：“叶子”还是“花朵”


下图是用数字来代表“叶子”和“花朵”的示例：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241026200011726-1702555556.png)


叶子的颜色由 RGB 值 (32, 107, 56\) 表示，体积为 11\.2 毫升。花朵的颜色由 RGB 值 (241, 200, 4\) 表示，体积为 59\.5 毫升。图中这些数据用于训练神经网络，让它学会根据“颜色”和“体积”来识别叶子和花朵。


现在我们就构建一个神经网络来完成这个分类任务。我们先确定输入/输出的格式和对输出结果的解释方式。上图中的叶子（Leaf）和花朵（Flower）已经用数字来表示了，所以可以直接传递给神经网络中的神经元。由于神经网络只能输出数字，因此我们还需要对神经网络输出的数字进行定义（即什么样的数字代表神经网络识别出的物体类型，是“叶子”还是“花朵”，因为神经网络本身无法直接输出“叶子”和“花朵”这两个名称来告诉我们分类结果）。因此，我们需要定义一个解释方案，将输出的数字对应到物体类别上：


• 如果设计的神经网络只有一个输出神经元，可以通过正数或负数来代表识别出的物体类别。当该神经元输出正数时，我们就认为神经网络识别出该物体是“叶子”；如果输出的是负数，则表示识别出的物体是“花朵”。


• 另外，也可以设计具有两个输出神经元的神经网络，用这两个神经元分别代表不同的物体类别。例如：约定第一个神经元代表“叶子”，第二个神经元代表“花朵”。当神经网络的第一个神经元输出的数字大于第二个神经元输出的数字时，我们就说神经网络识别出当前的物体是“叶子”；反之，当第一个神经元输出的数字小于第二个神经元输出的数字时，我们就说神经网络识别出的当前物体是“花朵”。


这两种方案都可以让神经网络识别物体是“叶子”还是“花朵”。但本文我们选择第二种方案，因为它的结构更容易适应后面我们要讲解的内容。以下是根据第二种方案设计出来的神经网络。让我们详细分析它：


![](https://img2024.cnblogs.com/blog/3524016/202410/3524016-20241026200200811-657562525.png)




---


图中的圆圈代表神经网络中的神经元，每一竖排代表网络的一层。所有的数据从第一层进入，然后逐层逐个做乘法和加法计算，经过隐藏层（三个神经元），最终到达输出层（两个神经元），我们根据最后一层的两个神经元输出的数值来预测当前识别出的是什么物体。注意图中的箭头和数字，以及它们之间的乘法与加法关系。


蓝色圆圈中的计算如下：(32 \* 0\.10\) \+ (107 \* \-0\.29\) \+ (56 \* \-0\.07\) \+ (11\.2 \* 0\.46\) \= \-26\.6


一些行话（专业名词）：


• 神经元/节点：带数字的圆圈


• 权重：箭头线上标注的数字


• 层：一组（排）神经元称为一层。可以将这个网络看作有三层：4个神经元的输入层、3个神经元的中间层和2个神经元的输出层。


要计算网络的预测/输出（称为“前向传播”），从左侧开始。把“叶子”代表的数字填入到第一层的神经元中。要前进到下一层，将圆圈中的数字与对应神经元的权重相乘并相加。我们演示了蓝色和橙色圆圈的计算。运行整个网络后，输出层第一个数字较大，因此我们可以解释为“网络将这些（RGB, Vol）值分类为叶子”。经过良好训练的网络可以处理各种（RGB, Vol）输入并正确分类物体。


这个神经网络模型本身对“叶子”、“花朵”或（RGB, Vol）没有任何概念（即它不理解“叶子”和“花朵”是什么）。它被设计出来只是为了接收4个数字并输出2个数字。我们规定4个输入数字代表物体的颜色值和体积，同时也规定2个输出神经元的值如何对应“叶子”和“花朵”。最终，网络的权重是通过训练过程自动调整得到的，以确保模型能够接收输入数字并输出符合我们解释的结果。


一个有趣的副作用是，也可以用这个神经网络来预测未来一小时的天气情况。我们将例如：云量和湿度等表示成4个不同的数字值作为输入，并将神经网络的最后输出解释为“1小时内晴天”或“1小时内下雨”。如果这个神经网络的权重校准良好，网络就可以同时完成分类叶子/花朵和预测天气的任务。我们的神经网络只是输出了两个数字，而这两个数字到底代表什么意思则完全取决于你对它的定义。例如：这两个数字可以代表对物体进行分类的结果或者预测天气等。


编写本小节时，为了让更多人理解，我省略了以下的一些技术术语。即使忽略这些术语，您依然可以理解神经网络的基本概念：


• 激活层：


神经网络通常有一层“激活层”，它对每个节点的计算结果应用一个非线性函数，以增强网络处理复杂情况的能力。一个常见的激活函数是 ReLU，它会将负数设为零，而正数保持不变。例如，在上例中，我们可以将隐藏层中的负数替换为零，然后再传递到下一层计算。没有激活层时，网络中的所有加法和乘法可以简化为单层。例如，绿色节点的输出可以直接写成 RGB 的加权和，不需要隐藏层。激活层的非线性特性使得神经网络能够处理更复杂的模式。


• 偏置：


神经网络中的每个节点通常还关联一个“偏置”值，这个值会加到节点的加权和结果中，用于调整输出。例如，如果顶层蓝色节点的偏置是 0\.25，那么计算公式变成：(32 \* 0\.10\) \+ (107 \* \-0\.29\) \+ (56 \* \-0\.07\) \+ (11\.2 \* 0\.46\) \+ 0\.25 \= \-26\.35。偏置使得网络可以更灵活地拟合数据，“参数”通常指模型中的这些权重和偏置值。


• Softmax：


在输出层，我们通常希望将结果转化为概率。Softmax 函数是一种常用的方法，它能将所有输出数值转换为概率分布（总和为 1）。Softmax 会将每个输出数值的指数除以所有输出值的指数和，使得输出层的结果可以被解读为各分类的概率。例如，如果 Softmax 处理后的值为 0\.8 和 0\.2，那么这表示 80% 的概率是“叶子”，20% 的概率是“花朵”。


未完待续…


 本博客参考[悠兔机场](https://xinnongbo.com)。转载请注明出处！
