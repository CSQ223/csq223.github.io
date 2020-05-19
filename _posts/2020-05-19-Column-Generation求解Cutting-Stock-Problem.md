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

# Cutting Stock Problem 问题描述

给定了若个根长度为$L$钢管，有$n$种钢管长度的订单，即长度为$l_i$的需求为$b_i$，求最少需要多少根这样的钢管才能满足需求？

在本文中，假定每根钢管长度$L=17m$，需要3米、6米、9米的钢管各25、20、18根，即$l = [3m,6m,9m]$，$b=[25,20,18]$。

# 整数规划模型
设给定$N$根长度为$L$的钢管，有$I$种钢管的需求，长度为$l_i$的需求量为$b_i$根。记$y_n$为0-1变量，如果第$n$根钢管被使用为1，否则为0。记$x_i^n$为第$n$根钢管被切了长度为$l_i$的长度的数量。由此，建立以下数学模型\ref{eq:obj}-\label{eq:y}：

\begin{equation}
\min\sum_{n\in N}y_n \label{eq:obj}\tag{1}
\end{equation}
S.T.
\begin{equation}
\sum_{n\in N}x_i^n\geqslant b_i, \forall i \in I \tag{2}
\end{equation}
\begin{equation}
\sum_{i=1}^{I}l_i x_i^n\leqslant Ly_n, \forall n\in N \tag{3}
\end{equation}
\begin{equation}
x_i^n\in \mathcal{Z}_+, \forall i \in I, \forall n\in N \tag{4}
\end{equation}
\begin{equation}
y_n\in\{0,1\}, \forall n\in N \label{eq:y}\tag{5}
\end{equation}


