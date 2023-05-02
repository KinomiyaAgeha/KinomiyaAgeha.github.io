---
title: 「字符串学习笔记」#2 Z函数（扩展KMP）
urls: notes-string-2
tags:
  - 字符串
  - 扩展KMP
categories: 学习笔记
math: true
abbrlink: eaef42cf
date: 2022-08-20 15:44:58
---

## 前言

本来想着，暑假要多学点算法，结果就学了一点点……

大部分时间拿来写题了。

有点小遗憾。

不过无妨。

<!--more-->

## Z 函数（扩展 KMP）

### 算法流程及实现

约定：字符串下标以 $0$ 为起点。

对于一个字符串 $S$，满足 $|S| = n$，它的 $z$ 函数定义为：$z(i)$ 表示 $S$ 和以 $i$ 开头的后缀 $S[i,n-1]$ 的最长公共前缀 (LCP) 的长度。

显然有 $z(0)=n$。

国外一般将计算该数组的算法称为 **Z Algorithm**，而国内则称其为**扩展 KMP**。

为啥是扩展 KMP？可能是因为这两段的 LCP，某种意义上也是一个 Border。

根据定义易得 $O(n^2)$ 的实现

```cpp
// C++ Version
vector<int> z_function_trivial(string s) {
    int n = (int)s.length();
    vector<int> z(n);
    z[0] = n;
    // 这句是我加的
    // OI-wiki上说z[0]=0，但是z[0]=0不仅说不过去而且板子题也过不了啊
    for (int i = 1; i < n; ++i)
    while (i + z[i] < n && s[z[i]] == s[i + z[i]]) ++z[i];
    return z;
}
// Code from oi-wiki.org
```

一般情况下，当一个能求出很 NB 的东西的算法复杂度过高时，就会有神犇来优化它。

>如同大多数字符串主题所介绍的算法，其关键在于，运用自动机的思想寻找限制条件下的状态转移函数，使得可以借助之前的状态来加速计算新的状态。

就像是 KMP 算法中的 $next$ 数组一样，这里考虑用 $z(0),z(1),\ldots z(i-1)$ 来求出 $z(i)$。

对于 $i$，我们称区间 $[i,i+z(i)-1]$ 是 $i$ 的**匹配段**，也可以叫 Z-box。

设 $[l,r]$ 为最靠右的 Z-box 对应的左右端点，必须保证 $l \le i$，初始化 $l=r=0$.

对于 $z(i)$，如果 $i \le r$，那么由于 $[l,r]$ 是一个匹配段，所以 $s[i,r] = s[i-l,r-l]$。

因此 $z(i) \ge \min \big(z(i-l),r-i+1 \big)$。

这时候

- 如果 $z(i-l) < r-i+1$，那么 $z(i)=z(i-l)$。
- 否则 $z(i-l) \ge r-i+1$，这时候令 $z(i) = \max(0,r-i+1 )$，然后暴力递增 $z(i)$，知道不能扩展为止。求出 $z(i)$ 后，如果 $r < i+z(i)-1$，那么就令 $l=i,r=i+z(i)-1$。

```cpp
void Z() {
	int l=0, r=0, ans=0;
	z[0]=n;
	for(int i=1;i<n;++i) {
		if(i<=r&&z[i-l]<r-i+1) z[i]=z[i-l];
		else {
			z[i]=max(r-i+1,0ll);
			while(i+z[i]<n&&s[z[i]]==s[i+z[i]]) ++z[i];
			if(r<i+z[i]-1) l=i, r=i+z[i]-1;
		}
	}
}
```

### 复杂度分析

对于内层`while`循环，每次都使得 $r$ 向后移动一位，而 $r \in [0,n-1]$，因此复杂度为 $O(n)$。

对于外层循环，只有一次遍历，复杂度 $O(n)$。

因此整个算法的复杂度为 $O(n)$。

## 应用



### 求出模式串与文本串每一个后缀的LCP

也就是洛谷上的模板题。

类似于 KMP 算法，求出模式串的 $z$ 函数，设文本串 $|T|=m$，那么设 $f(i)$ 为模式串与文本串的后缀 $T[i,m-1]$ 的 LCP 长度，在 $z$ 函数的基础上处理即可。

```cpp
void exkmp() {
	Z();
	int l=-1, r=-1, ans=0;
    // 注意从l=r=-1
	for(int i=0;i<m;++i) {
		if(i<=r&&z[i-l]<r-i+1) f[i]=z[i-l];
        // 这里是z[i-l]
		else {
			f[i]=max(r-i+1,0ll);
			while(i+f[i]<m&&s[f[i]]==t[i+f[i]]) ++f[i];	
			if(r<i+f[i]-1) l=i, r=i+f[i]-1;
		}
	}
}
```

### 匹配所有子串

寻找模式串 $P$ 在文本串 $T$ 中的所有出现 (occurrence)。

构造字符串 $S = P + \lambda + T$，其中 $\lambda$ 是一个不在二者之中出现的字符。

首先计算出 $S$ 的 $z$ 函数，对于任意 $i \in [0,|T|-1]$，考虑以 $T_i$ 开头的后缀在 $S$ 中的函数值 $k = z(i+|S|+1)$，如果 $k = |S|$，那么说明有一个 $P$ 出现在了 $T$ 的第 $i$ 个位置。

不难发现此种方法也可以求解上面的那个问题。



### 字符串整周期

找到给定字符串 $|S|=n$ 的最小整周期。

考虑 $S$ 的 $z$ 函数，则其最小整周期为满足 $i \mid n$ 且 $i+z_i = n$。

证明？感性理解~



&nbsp;

嗯嗯，就写这么点吧。

由于时间原因，本文没有放很多证明过程。

## 参考

- [OI-wiki Z 函数（扩展 KMP）](https://oi-wiki.org/string/z-func/)
