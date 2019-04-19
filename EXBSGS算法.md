---
title: EXBSGS算法
date: 2019-04-19 11:43:46
categories: 算法
tags:
- 数论
- EXBSGS
mathjax: true
---

EXBSGS

<!-- more -->

$a^{x} \equiv b(\bmod p)$

BSGS只能求解p为素数的情况，EXBSGS可以完美解决这个问题

# EXBSGS

参考：[https://www.cnblogs.com/TheRoadToTheGold/p/8478697.html](https://www.cnblogs.com/TheRoadToTheGold/p/8478697.html)

求解$a^{x} \equiv b(\bmod p)$（P不一定是质数）的最小非负正整数解

**先放三个同余定理**

- 定理1：

![](EXBSGS算法/gongshi1.png)

- 定理2：

![](EXBSGS算法/gongshi2.png)

- 定理3：

![](EXBSGS算法/gongshi3.png)

**求解**

1. 如果b==1，那么x=0，算法结束

2. 若gcd(a, p) != 1，令d=gcd(a, p)，若d不能整除b，则无解，算法结束

   ```
   例如当x=1，a=4，p=8，b=3时，代入公式有4 mod 8和3 mod 8，此时d = gcd(a, p) = 4，说明a与p有因子为4，但是d不能整除b，说明b中没有共同因子，同样mod同一个数，没有共同因子，那么说明方程无解
   ```

3. 把一个a提取出来，$a^{x} \equiv b(\bmod p)$变成$a*a^{x-1} \equiv b(\bmod p)$

   同时除以d得