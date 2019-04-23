## 笔记

## ASE14Mining API mapping for language migration

### Keywords

API Mapping , Code Migration , API Usages , Statistical Learning 

主要学习采用方法：`统计学习`

### 一些生词

**corpus**: a collection of written texts, especially the entire works of a particular author or a body of writing on a particular subject. 文集

**heuristic**：启发式

### 专有词汇

`Groum` : 用来表示 `API usages in the method`

`usages symbols`: The labels extracted from nodes' labels （提取出来的用法符号）

`API usages`: 一系列的usages symbols

`API usages Mapping`: mapping on the programming tasks

`API Mapping`: Usage contains only one class / method



### Introduction

###### code migration

软件迁移。这里的软件是广义的软件，可以小到一个程序

###### 一些问题

一对多映射；方法参数；方法与字段的映射

###### 得到数据的方式

- a tool search for the respective methods **in the pairs of** corresponding implementations in Java and C#

  Same method signatures / same or similar names



### Motivating Example

- Java version

```java
1 HashMap<String, Integer> myVocabIdxDict = new HashMap<String,Integer>();
2 myVocabIdxDict.put("alphabet", 1);
3 FileWriter writer = new FileWriter("VocabIdx.txt") ;
4 for (String vocab: myVocabIdxDict.keySet()){
5  	writer.append(vocab + " "+ myVocabIdxDict.get(vocab)+"nrnn");
6 }
7 writer.close () ;
```

包含了三个`API usages`:

1. vocabulary instantiation & populate

2. #### reading data & loop

3. write data out & close resource

- C# version

```c#
1 Dictionary<string, int> myVocabIdxDict = new Dictionary<string,int>();
2 myVocabIdxDict.Add("alphabet", 1);
3 StreamWriter writer = new StreamWriter("VocabIdx.txt");
4 List<string> keyList = new List<string>(myVocabIdxDict.Keys);
5 foreach( string vocab in keyList ){
6 	int idx ;
7 	myVocabIdxDict.TryGetValue(vocab, out idx);
8	writer .WriteLine(vocab + " " + idx);
9 }
10 writer.Close() ;
```

当下的启发式挖掘，面临问题：

1. API elements 名称不一致
2. API elements 参数不一致
3. 一个API可能映射到另一个语言的字段
4. 一个API映射到另一个语言的多个API

### Mining Of API Usage Mappings

API Usages: 涉及到了类使用(引用)，方法调用、操作符重载、控制流

Groum: 用于表示图, for the symbol of the API usages

##### 算法执行流程

- 构建sub-groum ; 得到usage sequence用法序列——得到Sentence——sentence作为输入，利用EM算法，基于symbol来进行极大似然估计，得到相关的模型——推广到用法序列



1. 根据代码文集，构建主 Groum

2. 根据该Groum ， 建立sub-groums.

   子图的提取，可以是一个或多个变量相关的

   用来表示一些更细小的api usage

3. For the Sub-groums

   建立usage sequence

   `sequence-based alignment`

4. 将如上的usage sequence整合，成为该方法的sentence

5. sentence作为输入，通过 `symbol-to-symbol alignment algorithm`, EM算法进行训练

### 建立、构造sub-groum , usage sequence

1. 分析方法与图Groum $G_M$

   收集方法中的变量

   对于函数调用，将会建立一个**临时变量**

2. 将节点放入集合$T$, 序关系为图中关系(否则采取代码中的出现顺序)

3. 提取sub - groum. 

   **标准程序分片**；包含控制流依赖

4. 通过子图中，==选取$k$个变量==，得到 usage sentences.

   并且使用标点符号

### Statistical Alignment

#### Symbol-to-symbol Alignment

假设我们已经得到了相应的`usage sequences`，

在两个不同的语言中 ,$s=s_1s_2s_3...s_m$ in $L_s$，以及$t=t_1t_2t_3...t_l\;in \; L_T$

###### Goal

Compute $P(s|t)$

```
The probability that 's' is the corresponding of t  GIVEN the obserable 't'
```

###### 步骤

1. $s$的长度$m$以概率$P(m|t)$被选取。对于每一个位置$i$，有一个`symbol` $t_j \;\in \;t $ ; 基于$t_j$生成了一个$s_i$

   我们称 $s_i$和$t_j$是**对齐(aligned)**的. 我们用$a_i = j$ 来进行表示 `s aligned with t`

   同时，$s_i$有可能不和 $t$ 中的任何一个`symbol`对齐，那么就使用 `null`表示

2. `vector` $a=(a_1,a_2,...,a_m)$ with value range $(0,l)$ 称作 `an alignment of s and t`

   $a_i = 0 $表示 $t$ 中没有合适的来对齐

3. 接下来计算 $P(s,t)$

   基于如下假设：

   - $s$的长度$m$仅依赖于$t$的长度$l$ . `That is`  $P(m|t) = \lambda(m,l)$
   - alignment $a_i = j $ 的选取，仅依赖于 $i , m , l$， `That is` $P(a_i|i,m,t)=\pi(j,i,m,l)$
   - symbol  $s_i =u$仅依赖于 $t_j = v$ ， `That is`  $P(s_i|t_{a_i},i,m,t)=\tau(s_i,t_{a_i})$

基于如上假设，我们得到
$$
P(s,a|t)=\lambda(m,l)\prod_{i=1}^m(\pi(a_i,i,m,l)\tau(s_i,t_{a_i}))
$$

###### 推导

$$
P(s,a|t)=P(s|a,t)P(a|t)\\
而 P(j|i,m,t)P(s_i|t_{j},i,m,t)=
$$

再得到：
$$
P(s|t)=\sum_aP(s,a|t)
$$

###### EM算法使用

```c
1 function EM Algorithm(Training corpus T)
2 	Initialize l;p; t uniformly
3 	repeat
4 		reset all counts to 0
5 		for each pair (s,t) in T:
6 			m = length(s), l = length(t)
7 			count(m,l)++, count(l)++
8 			for i = 1..m, j = 0..l:
9 				u = si, v = t j
10 				p(i,j) = p(j,i,m,l) * t(u,v)
11 				d(i,j) = p(i,j)/ål
				j=0p(i,j)
12 				count(j,i,m,l) += d(i,j), count(i,m,l) += d(i,j)
13 				count(u,v) += d(i,j), count(v) += d(i,j)
14 			for all m,l:
15 				l(m,l) = count(m,l)/count(l)
16 			for all j,i,m,l:
17 				p(j,i,m,l) = count(j,i,m,l)/count(i,m,l)
18 			for all u,v:
19 				t(u,v) = count(u,v)/count(v)
20 		until convergence
21 	return (l;p; t)
```

1. 首先随机化参数

2.  lines 3 - 20

   1. lines 4 - 13  Expectation Step

      使用了估计的参数($\lambda,\pi,\tau$)来推测对齐($s,t$)

      得到对齐的count

   2. lines 14 - 19 maximization step

      参数更新; 根据count的值大小，改变参数$\lambda,\pi,\tau$



#### Sequence-to-Sequence Alignment

`phrase-based model` 基于**短语**的模型

1. 模型将 `基于符号的` 对齐的symbol和相关**概率**加入sequence mapping table

2. 收集序列对(和符号对齐相互对应)——也就是说，序列对应需要包含所有的符号对应

   eg.:	

若所有的符号$s_1,s_2,...,s_k$，拥有他们的对应点于$t_1,t_2,...,t_k$，那么我们称序列对($s,t$)是对应(consistent)的

3. 遍历所有的目的序列，找到和源序列最接近的；把序列对和映射概率添加到表中
4. 过滤：留下高于阈值的序列对