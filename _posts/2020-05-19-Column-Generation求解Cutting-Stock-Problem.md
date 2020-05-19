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

##  主问题
我们首先确定主问题的模型。设当前一共有P种切割方案，第$p$种方案钢管切长度为$l_i$的切割数量为$a_{ip}$，第$p$种方案出现了$z_p$次。由此，主问题模型为：
\begin{equation}
\min \sum_{p\in P}z_p \label{eq:rmpobj}\tag{6}
\end{equation}
subject to:
\begin{equation}
\sum_{p\in P}a_{ip}z_p\geqslant b_i, \forall i\in I \label{eq:rmpdemand}\tag{7}
\end{equation}
\begin{equation}
z_p\geqslant 0,\forall p\in P \label{eq:rmpz}\tag{6}
\end{equation}


