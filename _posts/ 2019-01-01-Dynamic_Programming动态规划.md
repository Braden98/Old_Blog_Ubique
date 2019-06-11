---
layout:     post
title:      "深入理解动态规划"
subtitle:   "两种写法、三个概念与若干题解"
date:       2019-01-23 12:31:14
author:     "Ubik"
header-img: "img/post-bg-halting.jpg"
catalog: true
comments: true
tags:
    - 算法
    - 动态规划
---



#  Leetcode Dynamic Programming 动态规划
## 两种写法
一开始，最基本的思路一般是暴力搜索（递归），从中发现重叠子问题，随后通过维护数据结构避免重复计算，此时即实现自顶向下的记忆化搜索。通过刚才的方法即可推导出动态规划。
动态规划最重要的有两点，状态的定义和状态转移方程。这两点确定了剩下的只是简单的编码（一般是几重循环）。
实际编码中，状态定义后数据结构可以进行化简，如背包问题和打家劫舍问题，只需要常数个变量即可。
### 递归
自顶向下，如对斐波那契数列的简化
### 递推
自底向上，如数塔问题的优化解
## 三个概念
用来思考解决新的dp问题的基础
### 重叠子问题
一个问题可以分解为若干的子问题，且这些子问题有重叠的部分。
### 最优子结构
一个问题的解可以由其子问题的解得到。（子问题的最优决策导出原问题的最优决策）
### 无后效性
子问题的解可由之前得到的子问题的解确定，而不依赖于之后还未解决的子问题。也就是说，前边影响后边，反之不成立。如未来可以看作现在关于过去的函数。
## 理解
dp实际上就是通过记录（维护若干数据结构）来避免重复求解重叠子问题。（空间换时间）这么做的前提是该问题有最优子结构，且无后效性。
#题解思考与记录
## 343
1.DP：`dp[n]=max(i*dp[n-i],i*(n-i))(其中i从1到n-1)`
边界条件是dp[2]=0,dp[3]=2;
2.数学推导，由基本不等式知，n个数的算术平均数大于等于它们的几何平均数且当且仅当各个数字相等时取等。为此设要分成的数字是x，f(x)=x^(n/x)，对其求导求得x=e时取最大，故取3；然后直接计算pow即可。注意到递归求解pow需要的时间复杂度为O(LogN)（主成分定理根据递归表达式可求得），O（logN)空间复杂度，。
另外附一个非递归实现pow
```
#include<iostream>
using namespace std;
 
template <class T,class Integer>
T power(T x,Integer n)
{
    if(n == 0) return 1;
    if(n == 1) return x;
    while((n&1) == 0)
    {
        x = x*x;
        n >>= 1;
    }
    T result = x;
    n >>= 1;
    while(n != 0)
    {
        x = x*x;
        if((n&1) != 0)
            result *= x;
        n >>= 1;
    }
    return result;
}
```
## 279
1.dp
`dp[n] = Min{ dp[n - i*i] + 1 },  n - i*i >=0 && i >= 1`
我们定义dp[i]为整数i最少能分解成多少个正整数的平方和。那么有，
`dp[i + j * j] = min(dp[i + j * j], dp[i] + 1).
`
如果一个数x可以表示为一个任意数a加上一个平方数bｘb，也就是x=a+bｘb，那么能组成这个数x最少的平方数个数，就是能组成a最少的平方数个数加上1（因为b*b已经是平方数了）。

```
public static int numSquares(int n) {
	int[] dp = new int[n + 1];
	// 将所有非平方数的结果置最大，保证之后比较的时候不被选中
	Arrays.fill(dp, Integer.MAX_VALUE);
	// 将所有平方数的结果置1
	for (int i = 0; i * i <= n; i++) {
		dp[i * i] = 1;
	}
	// 从小到大找任意数a
	for (int a = 0; a <= n; a++) {
		// 从小到大找平方数bｘb
		for (int b = 0; a + b * b <= n; b++) {
			// 因为a+b*b可能本身就是平方数，所以我们要取两个中较小的
			dp[a + b * b] = Math.min(dp[a] + 1, dp[a + b * b]);
			System.out.println(a + b * b + ":--" + Arrays.toString(dp));
		}
	}
	return dp[n];
}

```
2.数学解法
![插图]({{site.baseurl}}/img/in-post/15539558523343.jpg)
根据四平方和定理，任意一个正整数均可表示为4个整数的平方和，其实是可以表示为4个以内的平方数之和，那么就是说返回结果只有1,2,3或4其中的一个，首先我们将数字化简一下，由于一个数如果含有因子4，那么我们可以把4都去掉，并不影响结果，比如2和8,3和12等等，返回的结果都相同，如果一个数除以8余7的话，那么肯定是由4个完全平方数组成。
```
public class Solution {
    public int numSquares(int n) {
		while (n % 4 == 0) n /= 4;
		if (n % 8 == 7) return 4;
		for (int a = 0; a * a < n; a++) {
			for (int b = 0; b * b <= n - a * a; b++) {
				if (a * a + b * b == n) {
					return (a > 0 ? 1 : 0) + (b > 0 ? 1 : 0);
				}
			}
		}
		return 3;
    }
}

```
## 509
###思路
最简单的记忆化搜索代替递归，注意从0开始，故new一个n+1长的dp数组。
### 代码
```
class Solution {
    public int fib(int N) {
        //注意从0开始，输入N，则有n+1项，故dp长度为n+1
        int[] dp=new int[N+1];
        if(N<=1)
            return N;
        dp[0]=0;
        dp[1]=1;
        for(int i=2;i<=N;i++)
        {
            dp[i]=dp[i-1]+dp[i-2];
        }
        return dp[N];
    }
}
```
## 70 爬楼梯
### 思路
要爬n阶楼梯，每次只能走一步或者两步，那么最终结果就分两种：从n-1阶走上来和从n-2阶走上来，由此得递推公式：`dp[n]=dp[n-1]+dp[n-2]`,注意到index存在n-2，故0，1需要特殊考虑
### 代码
```
class Solution {
    public int climbStairs(int n) {
        if(n==1)
            return 1;
         int dp[]=new int[n+1];
        dp[0]=1;//为使index=2时符合递推
        dp[1]=1;
        for(int i=2;i<=n;i++){
            dp[i]=dp[i-1]+dp[i-2];
        }
        return dp[n];
    }
}
```
## 746
### 思路
题目有一个坑点（根据正确解答推断出），数组下标从0开始，但是楼梯层数默认从1开始，这里要对应。
和上一题一样，到n层的最少花费只需要考虑从n-1层或者n-2层来加上n层的即可`dp[i]=Math.min(dp[i-1],dp[i-2])+cost[i];
`。边界条件是`dp[i]=cost[i],i=1或者2`。
###  代码
```
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int len=cost.length;
        int dp[]=new int[len];
        if(len==1)
            return cost[0];
        dp[0]=cost[0];
        dp[1]=cost[1];
        for(int i=2;i<len;i++){
            dp[i]=Math.min(dp[i-1],dp[i-2])+cost[i];
        }
        //1,1,2,3,3,103,4,5,104,6
        //0,0,1,1->0,0,1,1
        //0,1,1,0->0,1,1,1
        //0,0,0,1->0,0,0,0,到倒数第二层时仍可选择上两层
        return Math.min(dp[len-1],dp[len-2]);
    }
}
```

##91 解码方法
### 思路
类似fib数列和爬楼梯，只是dp[n-1]不需加上，比如11，第二位解为单位字母A并不增加个数，只有dp[n-2]才会额外增加。此时的条件是数字在1～26之间。
一开始有一点疑惑，会不会有重叠？其实不会，因为这是子问题，不同的子问题之间（这里是dp[n-2]和dp[n-1]）是不同的问题，前边方法可能是重叠的，但因为问题不同，结果也就不会重叠。
### 代码
```
public class Solution {
    public int numDecodings(String s) {
        if (s.isEmpty() || (s.length() > 1 && s.charAt(0) == '0')) return 0;
        int[] dp = new int[s.length() + 1];
        dp[0] = 1;
        for (int i = 1; i < dp.length; ++i) {
            dp[i] = (s.charAt(i - 1) == '0') ? 0 : dp[i - 1];
            if (i > 1 && (s.charAt(i - 2) == '1' || (s.charAt(i - 2) == '2' && s.charAt(i - 1) <= '6'))) {
                dp[i] += dp[i - 2];
            }
        }
        return dp[s.length()];
    }
}
```
## 62 机器人 不同路径
### 思路
和爬楼梯一样，那里的一部or两部to这里的向下或者向右走，这是最基本的动态规划解法。还有数学解法，机器人一定会走m+n-2步，即从m+n-2中挑出m-1步向下走即可。
### 
### 代码
dp
```
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] dp=new int[m][n];
        for(int i=0;i<m;i++)
            dp[i][0]=1;
        for(int i=0;i<n;i++)
            dp[0][i]=1;
        for(int i=1;i<m;i++)
            for(int j=1;j<n;j++)
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
        return dp[m-1][n-1];
    }
}
```
数学解法（java报超时了）
```
class Solution {
public:
    int uniquePaths(int m, int n) {
        double num = 1, denom = 1;
        int small = m > n ? n : m;
        for (int i = 1; i <= small - 1; ++i) {
            num *= m + n - 1 - i;
            denom *= i;
        }
        return (int)(num / denom);
    }
};
```
## 300 LIS 最长上升子序列
###思路
最蠢的办法当然是枚举所有的子序列并判断是否上升，但从中很容易发现重复计算（也即重叠子问题：i~j的部分一旦算好了，i~j+1的部分就不必从头算起。也就是说，存在重叠子问题，且满足最优子结构，即上边说的原问题的解可由其部分解推出。由此可得状态转移方程`dp[i]=nums[i]>nums[i-1]?dp[i-1]+1:0,i>1&&i<nums.length`,边界条件为`dp[0]=0;`
稍微一试发现很蠢，只考虑了之前一个数（应该考虑之前所有的数字，没考虑到的原因是停留在之前某个题状态，也没有手动算一下），实际上应该考虑i之前所有的情况，故更新状态转移方程为
`dp[i]=max(dp[j]+(nums[i]>nums[j]?1:0),dp[i]),0<i<nums.length`
### 代码
出现了两个问题，一是选择运算符的优先级，二是代码中注释问题，如下。
```
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] dp=new int[nums.length];
        for(int i=0;i<nums.length;i++){
            dp[i]=1;
        }
        for(int i=0;i<nums.length;i++){
            int max=1;
            for(int j=0;j<i;j++){
                //dp[i]=Math.max(dp[i],dp[j]+(nums[i]>nums[j]?1:0));
                if(nums[i]>nums[j])
                    //max=Math.max(dp[i],dp[j]+1);
                    //这里max的第一个变量不应该是dp[i]，因为最大的变量始终记录在max中
                    //也就是说，对于max中包含迭代变量的，需要一个max_flag来记录
                    max=Math.max(max,dp[j]+1);
            }
            dp[i]=max;
        }
        int ans=0;
        for(int i=0;i<nums.length;i++){
            ans=Math.max(ans,dp[i]);
        }
        return ans;
    }
}
```
### 优化 
上述问题的时间复杂度在O(n^2），**可以通过一些二分查找优化到O(n*logn)**
**也可以通过树状数组优化**https://www.cnblogs.com/acmsong/p/7231069.html
### 深入
如果要找出这个LIS，应该怎么办？

##376 摆动序列 
###思路
和LIS问题一样，避免了枚举这种愚蠢且**错误**的方法，而是考虑已经解决i～j范围的问题，则i～j+1的问题答案可得出（思路和LIS一样），由此得到状态转移方程为`dp[i]=max(dp[i],dp[j]+(nums[i]-nums[....`注意到dp[j]+后边的判断需要记录之前的是增是减，故此种方法需要结构体（内部类）对这个信息进行储存，故优先考虑别的方法。






