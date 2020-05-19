---
layout:     post   				    # 使用的布局（不需要改）
title:      Column Generation求解Cutting Stock Problem		    # 标题 
subtitle:   列生成，Cutting Stock Problem，GUROBI #副标题
date:       2020-05-17 				# 时间
author:     Lewis XU 					# 作者
header-img: /img/post-bg-rwd.jpg #这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 运筹优化
    - 列生成
    - GUROBI
---

# 引言

列生成算法是一种解决大规模整数规划的框架，比直接使用商业求解器求解模型的速度要快。再次框架下，将问题分解为主问题（Restricted Master Problem，RMP）和子问题（SubProblem，SP）。子问题是求解一种可行的子方案，主问题则用来求解原问题的目标，而子问题的目标则是主问题的检验数。一个子问题的解，能够添加进主问题，对最小化问题而言，子问题的目标函数必须为负。列生成就是不断的求解子问题，把子问题逐个的添加进主问题中，直到子问题的解不满足条件。

# Cutting Stock Problem 问题描述

给定了若个根长度为$L$钢管，有$n$种钢管长度的订单，即长度为$l_i$的需求为$b_i$，求最少需要多少根这样的钢管才能满足需求？

在本文中，假定每根钢管长度$L=17m$，需要3米、6米、9米的钢管各25、20、18根，即$l = [3m,6m,9m]$，$b=[25,20,18]$。

# 整数规划模型
设给定$N$根长度为$L$的钢管，有$I$种钢管的需求，长度为$l_i$的需求量为$b_i$根。记$y_n$为0-1变量，如果第$n$根钢管被使用为1，否则为0。记$x_i^n$为第$n$根钢管被切了长度为$l_i$的长度的数量。由此，建立以下数学模型(\ref{eq:obj})-(\ref{eq:y})：

\begin{equation}
\min\sum_{n\in N}y_n \label{eq:obj}\tag{1}
\end{equation}
subject to:
\begin{equation}
\sum_{n\in N}x_i^n\geqslant b_i, \forall i \in I \label{eq:demand}\tag{2}
\end{equation}
\begin{equation}
\sum_{i=1}^{I}l_i x_i^n\leqslant Ly_n, \forall n\in N \label{eq:length}\tag{3}
\end{equation}
\begin{equation}
x_i^n\in \mathcal{Z}_+, \forall i \in I, \forall n\in N \label{eq:x}\tag{4}
\end{equation}
\begin{equation}
y_n\in\{0,1\}, \forall n\in N \label{eq:y}\tag{5}
\end{equation}

公式（\ref{eq:obj}）表示最小化钢管数量，公式（\ref{eq:demand}）表示每种钢管的数量得到满足，公式（\ref{eq:length}）表示每根钢管的切割的长度总和不超过$L$.(\ref{eq:x})-(\ref{eq:y})则定义了变量$x_i^n$和$y_n$的定义域。

# Column Generation
该问题可以等价于以下描述：一根钢管有不同的切割方案，比如17$m$可以切成5根3$m$的，也可以切成1根3$m$、2根6$m$的...。我们的目标是从这些方案中，选出消耗钢材数量最小的组合，以满足各个长度的数量。

我们首先确定主问题的模型。设当前一共有P种切割方案，第$p$种方案钢管切长度为$l_i$的切割数量为$a_{ip}$，第$p$种方案出现了$z_p$次。由此，主问题模型为公式（\ref{eq:mpobj}）-（\ref{eq:mpz}）：
\begin{equation}
\min \sum_{p\in P}z_p \label{eq:mpobj}\tag{6}
\end{equation}
subject to:
\begin{equation}
\sum_{p\in P}a_{ip}z_p\geqslant b_i, \forall i\in I \label{eq:mpdemand}\tag{7}
\end{equation}
\begin{equation}
z_p\in \mathcal{Z}_+,\forall p\in P \label{eq:mpz}\tag{6}
\end{equation}

公式（\ref{eq:mpobj}）是使钢管数量最小,公式(\ref{eq:mpdemand})则是满足所有的需求。

##  主问题
一次性找出所有的满足条件的模式是不现实的，我们先给定一些初始可行解$P'$，能够满足所有的需求，在通过子问题，不断求出新的可行方案，添加到$P'$中，直到达到最优解即可。
\begin{equation}
\min \sum_{p\in P'}z_p \label{eq:rmpobj}\tag{6}
\end{equation}
subject to:
\begin{equation}
\sum_{p\in P'}a_{ip}z_p\geqslant b_i, \forall i\in I \label{eq:rmpdemand}\tag{7}
\end{equation}
\begin{equation}
z_p\geqslant 0,\forall p\in P' \label{eq:rmpz}\tag{8}
\end{equation}

>如何寻找初始解？

显然，这种切割方式过于粗糙。下面，通过子问题来生成更多的可行方案，将这些粗糙的方案踢出去（实际上就是单纯形法的出基）。

## 子问题
子问题是求一种可行的切割方案，使得把这种方案添加进去之后（实际上就是单纯形法的入基），主问题的目标函数能下降最多，所以正如引言所说，子问题的目标函数是使主问题的校验数最小。设$\pi_i$是主问题中第$i$个约束的对偶值，所以，子问题的模型如下： 

\begin{equation}
\min 1-\sum_{i\in I}a_{ip}\pi_i \label{eq:spobj}\tag{9}
\end{equation}
subject to:
\begin{equation}
\sum_{i\in I}l_i a_{ip} \leqslant L \label{eq:splength}\tag{10}
\end{equation}
\begin{equation}
a_{ip}\in\mathcal{Z}_+, \forall i\in I \label{eq:spa}\tag{11}
\end{equation}

公式（\ref{eq:splength}）就是限制切割方案的长度，目标函数(\ref{eq:spobj})则是主问题的检验数。

不断的进行迭代，直到子问题的解大于0（对于最小化问题），就可以结束了。


# 算例

## 寻找初始解
只要将每根钢管按照同一种长度来切，一定能切出满足条件的组合。

所以初始解的方式：

1. 全切成3m以满足3m的需求，一根可以切出$\lfloor\frac{17}{3}\rfloor$= 5根;
2. 全切成6m以满足6m的需求，一根可以切出$\lfloor\frac{17}{6}\rfloor$= 2根;
3. 全切成9m以满足9m的需求，一根可以切出$\lfloor\frac{17}{9}\rfloor$= 1根。

也就是说，初始主问题模型：
\begin{equation}
\min z_1 + z_2 + z_3
\end{equation}
\begin{equation}
5z_1 + 0z_2 + 0z_3 \leqslant 25
\end{equation}
\begin{equation}
0z_1 + 2z_2 + 0z_3 \leqslant 20
\end{equation}
\begin{equation}
0z_1 + 0z_2 + 1z_3 \leqslant 18
\end{equation}
\begin{equation}
z_1, z_2, z_3 \geqslant 0
\end{equation}

