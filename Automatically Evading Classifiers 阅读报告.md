# 论文链接
https://pdfs.semanticscholar.org/5e4f/a9397c18062b970910f8ee168d3297cf098f.pdf 
# 论文的核心
1. 提出使用基因编程的方法，寻找恶意PDF的对抗逃逸样本。对抗逃逸样本有两个要求: 1. 样本依然是恶意的，样本的某些恶意行为没有改变 2. 样本被分类器错误的识别为Benign样本。
2. 遗传算法寻找恶意PDF的过程：
```
- First, we initialize a population of variants by performing
random manipulations on the malicious seed. Then, each
variant is evaluated by a target classifier as well as an oracle.
The target classifier is a black box that outputs a number that
is a measure of predicted maliciousness of an input sample.
There is a prescribed threshold used to decide if it is malicious
or benign. The oracle is used to determine if a given sample
exhibits particular malicious behavior. In most instantiations,
the oracle will involve expensive dynamic tests.

- A variant that is classified as benign by the target classifier,
but found to be malicious by the oracle, is a successful evasive
sample. If no evasive samples are found in the population,
a subset of the generated variants are selected for the next
generation based on a fitness measure designed to reflect
progress towards finding an evasive sample. Since it is unlikely
that the transformations will re-introduce malicious behaviors
into a variant, corrupted variants that have lost the malicious
behavior are replaced with other variants or the original seed.
Next, the selected variants are randomly manipulated by
mutation operators to produce next generation of the population. 
The process continues until an evasive sample is found
or a threshold number of generations is reached.

- To improve the efficiency of the search, we collect traces
of the mutation operations used and reuse effective traces. If
a search ends up finding any evasive variants, the mutation
traces on the evasive variants will be stored as successful
traces. Otherwise, the mutation trace of a variant with the
highest fitness score is stored. These traces are then applied to
other malware seeds to generate variants for their population
initialization. Because of the structure of PDFs and the nature
of the mutation operators, the same sequence of mutations can
often be applied effectively to many initial seeds.
```
首先对恶意样本种子进行随机修改。然后每一次修改PDF文件的object:要么删除、要么添加、要么替换一个object.
新加入的object就从benign样本里面提取的。

3. 论文使用Cuckoo沙箱来检测特定的pdf文件，是否具有恶意行为。检测是否具有恶意行为其实也是看这个pdf是否
在动态运行的时候产生特定的HTTP Request和Host Request行为。沙箱并不能直接告诉用户该pdf是否恶意、良性，需要
用户自行根据沙箱的动态运行日志进行建模。

# 论文的核心贡献
1. 直接对PDF进行修改的这种逃逸攻击，不依赖与分类器类型和使用的特征。 这种方法与基于特征空间的方法具有很大的不同。
2. 因为直接对PDF进行修改，于是寻找对抗的过程使用并没有使用基于梯度的方法，而是使用遗传算法。当然是需要黑盒访问分类器的分类结果
来指导遗传算法模型的运行。
3. 论文揭示了一个很严重的问题： 我们所使用的机器学习模型，看似在训练集和测试集都取得很好的准确率。但是这并不能说明机器学习模型
学习到相应问题的本质。就如同恶意PDF检测问题里面使用的特征其实是很shallow的，这些特征可能跟PDF是否真的具有恶意行为（是否会远程下载可执行文件··)
没有任何联系，这些特征是人工提取的，它们充其量是在统计角度能够比较好的区分Benign和malware。 因此要让模型真的学习到问题本质，可能的方向：
- A. 增加样本，样本库完整以后，迫使模型不得不放弃shallow特征。
- B. 人工设计特征的时候，需要考虑那些与问题本质更加深层次的特征。
- C. 使用端到端的神经网络自动提取特征。但是神经网络也有可能会优先考虑那些浅层特征，此时应该人为的为模型加上约束。把那种问题相关的深层次特征编码到网络之中。
