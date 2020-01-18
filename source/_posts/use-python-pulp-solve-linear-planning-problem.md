---
title: 用Python的pulp库求解线性规划问题
date: 2020-01-19 00:05:47
excerpt: 线性规划（Linear planning 或者 Linear programming）是最常见的优化技术，在运筹学（operation research）、规划（planning）等领域有很多的应用场景。虽然不像机器学习那样被大肆宣传，但是对可以通过具有线性关系的决策变量来表达的问题而言却是最首选的方法。
categories:
- 算法
tags: 
- 线性规划
- Python
---

线性规划（Linear planning 或者 Linear programming）是最常见的优化技术，在运筹学（operation research）、规划（planning）等领域有很多的应用场景。虽然不像机器学习那样被大肆宣传，但是对可以通过具有线性关系的决策变量来表达的问题而言却是最首选的方法。

[《数据、模型与决策：基于电子表格的建模和案例研究方法》](https://book.douban.com/subject/26731100/)介绍了通过 Excel 来求解线性规划问题，本文介绍如何通过 Python 的 pulp 库来进行求解。

### 问题描述

案例来源于[《数据、模型与决策：基于电子表格的建模和案例研究方法》](https://book.douban.com/subject/26731100/)中第3章的超级食品公司的广告组合问题。

该公司雇佣了一家广告公司来帮助设计全国性促销活动，**以得到更多消费者的认可**，该公司将根据广告公司所提供的服务给予一定的酬金，酬金预算不超过100万美元，投放的广告预算不超过400万美元。有三种广告媒介可以选择：周六增刊广告、杂志广告、电视广告。由于电视广告资源有限，只有5个时段可以投放。

广告成本如下：（单位：千美元）

| 成本分类 | 每次电视广告 | 每次杂志广告 | 每次周日增刊广告 |
|---|---|---|---|
| 广告预算 | 300 | 150 | 100 |
| 规划预算 | 90 | 30 | 40 |

广告媒介受众数量：（单位：千）

|  | 每次电视广告 | 每次杂志广告 | 每次周六增刊广告 |
| --- | --- | --- | --- |
| 广告受众期望值 | 1300 | 600 | 500 |

解决一个线性规划，有如下三步：
1. 找出决策变量（decision variable）
2. 确定约束条件（functional constraint）
3. 目标函数（objective function）

### 安装和起步 

通过 pip 安装 pulp：
```shell
pip install pulp
```

首先要创建一个`LpProblem`对象：

```python
from pulp import *

problem = LpProblem("Ads Optimization", LpMaximize)
```

第二个参数`LpMaximize`表示这个问题属于最大化问题。相应地，`LpMinimize` 则表示这个问题属于最小化问题。

`LpProblem` 就像一个容器，后面会将 __决策变量__、 __约束条件__ 和 __目标函数__ 都放到这个容器中。

### 决策变量

第二步是定义决策变量，pulp 中定义决策变量的方式：

```python
variable = LpVariable("variableName")
```

默认是不设置上限和下限的，如果需要指定这个变量的上限和下限：

```python
var = LpVariable(name, lowBound=None, upBound=None, cat='Continuous', e=None)
```

`cat`表示Category，有三种：Integer, Binary 和 Continuous，默认是 Continuous。

如果决策变量比较多，而且这些决策变量属于同一种类型，也有相同的上限和下限，那么可以使用`dicts()`方法来统一创建：

```python
varDict = dicts(name, indexs, lowBound=None, upBound=None, cat='Continuous', indexStart=[])
```

这个例子中，一共有三个 决策变量，其中电视广告投放量是有上限约束的，其他两个则没有：

```python
tv = LpVariable("TV", lowBound=0, upBound=5)
m = LpVariable("Magazine", lowBound=0)
ss = LpVariable("SS", lowBound=0)
```

### 约束条件

接着使用决策变量来设置约束条件，这个例子中一共有两个约束条件：

```python
c1 = 300 * tv + 150 * m + 100 * ss <= 4000
c2 = 90 * tv + 30 * m + 40 * ss <= 1000

problem += c1
problem += c2
```

注意 `c1` 和 `c2` 实际上是一个表达式，最后则通过 `+=` 将两个约束条件添加到 `LpProblem` 对象中。

### 目标函数

这个例子中，目标函数是广告整体的受众数量，可以通过第二个表格计算：

```python
obj = 1300 * tv + 600 * m + 500 * ss

problem += obj
```

最后仍然是通过 `+=` 将目标函数添加到 `LpProblem` 对象中。

如果要看下整个线性规划问题的定义：

```python
print(problem)
```

输出如下，非常容易理解：

```
AdsOptimization:
MAXIMIZE
600*Magazine + 500*SS + 1300*TV + 0
SUBJECT TO
_C1: 150 Magazine + 100 SS + 300 TV <= 4000

_C2: 30 Magazine + 40 SS + 90 TV <= 1000

VARIABLES
Magazine free Continuous
SS free Continuous
TV <= 5 Continuous
```

### 求解

准备好之后，通过调用 `solve()` 方法进行求解，返回 1 表示问题解决了，返回 -1 表示无法求解。

```python
problem.solve()
```

通过`LpVariable.varValue`属性来获取找到的最优解：

```python
print(tv.name + " : " + str(tv.varValue))
print(m.name + " : " + str(m.varValue))
print(ss.name + " : " + str(ss.varValue))
```

输出的最优解为：

```
TV : 0.0
Magazine : 20.0
SS : 10.0
```

即不采用任何电视广告、投放20次杂志广告、投放10次周六增刊广告。

`pulp.value()` 函数用于获取最终的目标函数值，这个例子中总的收益为：

```python
total_user = int(value(problem.objective))
print("Total users: " + str(total_user))
```

输出为：
```
Total users: 17000
```
也就是说这个广告投放组合可以带来17000千人次的广告受众量。

### 参考

* [Pulp函数库参考](https://pythonhosted.org/PuLP/pulp.html)
* [Linear Programming in Python](https://hackernoon.com/linear-programming-in-python-a-straight-forward-tutorial-a0d152618121)
* [Linear programming and discrete optimization with Python using PuLP](https://towardsdatascience.com/linear-programming-and-discrete-optimization-with-python-using-pulp-449f3c5f6e99)