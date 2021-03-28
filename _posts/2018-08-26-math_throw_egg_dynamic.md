---
layout: default
title: 扔鸡蛋问题 动态规划大法
#excerpt: 
---
　　之前有一篇文章“扔鸡蛋问题”，写的是指定鸡蛋个数在指定楼层，求最优解。里面列举了二分法、平方根法和解方程法，但是，如果鸡蛋个数和楼层数是待定的，那这三种方法都搞不定了。所以，这里又得引申出新的方法“动态规划大法”

　　首先说一下什么是动态规划。动态规划，英文Dynamic programming，是求解决策过程最优化的数学方法。动态规划的大致思路是把一个复杂问题转化成一个分阶段逐步递推的过程，从简单的厨师状态一步一步递推，最终得到复杂问题的最优解。

　　动态规划分为两步：

      　　1. 寻找状态转移方程式
         2. 利用状态转移方程式自下而上的求解问题
　　 那如何找到 状态转移方程式呢？我们可以把M层楼/N个鸡蛋的问题转化成一个函数F(M,N)，其中，楼层M和鸡蛋N是两个参数，而函数的值则是最优解的最大尝试次数。假设我们第一个鸡蛋扔出的位置在第X层(1<=x<=m)，会出现两种情况：

1.第一个鸡蛋没碎

那么剩下的资源是  M-X 层楼，N个鸡蛋，可以转变为下面的函数：

F(M-X,N) + 1 ,1<=X<=M

2.第一个鸡蛋碎了

那么剩下的资源是 X-1 层楼，N-1个鸡蛋，可以转变成下面的函数：

F(X-1,N-1) + 1 ,1<=X<=M

　　也就是说，我们要求出M层楼/N个鸡蛋，最大尝试次数的最小值的解，可以用下面的状态转移方程式：

F(M,N) = Min( Max(F(M-X,N) + 1 , F(X-1,N-1) + 1) )  ,1<=X<=M

　　好了，第一步的“寻找状态转移方程式”搞定了，接下来是第二步“利用状态转移方程式来自下而上的求解”。根据动态规划的思想，我们需要从一个鸡蛋一层楼的最优尝试次数，一步一步推导后续的状态。

　　为了更加形象具体的表述推导的过程，这里使用表格法

<table>
    <theader>
        <tr>
            <td>鸡蛋个数\楼层个数</td>
            <td>一层楼</td>
            <td>二层楼</td>
            <td>三层楼</td>
            <td>四层楼</td>
        </tr>
	</theader>
	<tbody>
        <tr>
            <td>一个鸡蛋</td>
        	<td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
        </tr>
        <tr>
            <td>二个鸡蛋</td>
        	<td>1</td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>三个鸡蛋</td>
        	<td>1</td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
	</tbody>
</table>

　　一个鸡蛋的情况下，最大尝试次数就是楼层的个数；楼层数固定是一层，那最大尝试次数就是恒定1。

　　当2个鸡蛋，2层楼的情况下，我们就需要代入状态转移方程式了

　　F(2,2) = Min( Max( F(M-1,N) + 1 , F(X-1 , N-1) + 1) ) = Min( Max( F(2-X,2) + 1 , F(X-1 , 2-1) + 1) ),1<=X<=2

因为X的取值是1或者2，我们需要对X的值逐一尝试：

当X=1时，

F(2,2) = Max( F(2-X,2) + 1 , F(X-1 , 2-1) + 1) 

           = Max( F(2-1,2) + 1 , F(1-1 , 2-1) + 1) 
    
           = Max( F(1,2) + 1 , F(0 , 2-1) + 1) 
    
           = Max(1+1 , 0+1)
    
           = 2

当X=2时，

F(2,2) = Max( F(2-X,2) + 1 , F(X-1 , 2-1) + 1) 

           = Max( F(2-2,2) + 1 , F(2-1 , 2-1) + 1) 
    
           = Max(F(0,2) + 1 , F(1,1) + 1)
    
           = Max(0+1 , 1+1)
    
           = 2

所以，无论第一个鸡蛋是从第一次扔，还是从第二层扔，结局都是尝试2次。

<table>
    <theader>
        <tr>
            <td>鸡蛋个数\楼层个数</td>
            <td>一层楼</td>
            <td>二层楼</td>
            <td>三层楼</td>
            <td>四层楼</td>
        </tr>
	</theader>
	<tbody>
        <tr>
            <td>一个鸡蛋</td>
        	<td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
        </tr>
        <tr>
            <td>二个鸡蛋</td>
        	<td>1</td>
            <td>2</td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>三个鸡蛋</td>
        	<td>1</td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
	</tbody>
</table>

以此类推，可以得到所有的<td>内的值

<table>
    <theader>
        <tr>
            <td>鸡蛋个数\楼层个数</td>
            <td>一层楼</td>
            <td>二层楼</td>
            <td>三层楼</td>
            <td>四层楼</td>
        </tr>
	</theader>
	<tbody>
        <tr>
            <td>一个鸡蛋</td>
        	<td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
        </tr>
        <tr>
            <td>二个鸡蛋</td>
        	<td>1</td>
            <td>2</td>
            <td>2</td>
            <td>3</td>
        </tr>
        <tr>
            <td>三个鸡蛋</td>
        	<td>1</td>
            <td>2</td>
            <td>2</td>
            <td>3</td>
        </tr>
	</tbody>
</table>

OK，我们使用表格大法，理解了动态规划的第二步操作。那么，我们在程序里，如何用代码实现呢？

```java
public class ThrowEggs {
    public static void main(String[] args) {
        int floorNum = 4;
        int eggNum = 3;

        System.out.println(getMinSteps(floorNum, eggNum));
    }

    public static int getMinSteps(int floorNum, int eggNum) {
        if (floorNum < 1 || eggNum < 1) {
            return 0;
        }
        // 定义表格
        int[][] table = new int[floorNum+1][eggNum+1];
        // 表格初始化
        for(int m=1;m<=floorNum;m++){// 楼层
            for(int n=1;n<=eggNum;n++){ // 蛋个数
                table[m][n] = m;
            }
        }

        for(int n=2;n<=eggNum;n++){
            for(int m=1;m<=floorNum;m++){

                // 枚举N个鸡蛋，在M层楼层，所有的可能，如果初始化的值大于计算的值，则计算的值替换原坐标的值
                for(int k=1;k<=m;k++){
                    table[m][n] = Math.min(table[m][n], 1 + Math.max(table[m-k][n], table[k-1][n-1]));
                }

            }
        }

        return table[floorNum][eggNum];
    }
}
```

上面的代码是实现了动态规划的需求，但是，时间复杂度是O(N * M * M)，又涉及到了二维数组，所以空间复杂度是O(M * N)。时间复杂度不好优化了，空间复杂度倒是可以再优化一下，根据 F(M,N) = Min( Max(F(M-X,N) + 1 , F(X-1,N-1) + 1) )，每一次自下而上的求解，都只和上一层和本层的值有关联，所有我们可以只需保存上一层的结果。

```java
    public static int getMinStepsOptimized(int floorNum, int eggNum) {
        if (floorNum < 1 || eggNum < 1) {
            return 0;
        }
        // 上一次的结果集，鸡蛋数-1，floorNum楼层 的最优尝试次数
        int[] preCache = new int[floorNum+1];
        // 当前的结果集，当前鸡蛋数，floorNum楼层数，最优尝试次数
        int[] currentCache = new int[floorNum+1];

        // 初始化currentCache的值
        for(int i=1;i<=floorNum;i++){
            currentCache[i] = i; 
        }

        for(int n=2;n<=eggNum;n++){
            // 把上一次的值，传递给preCache
            preCache = currentCache.clone();
            // 重新初始化currentCache
            for(int i=1;i<=floorNum;i++){
                currentCache[i] = i;
            }

            for(int m=1;m<=floorNum;m++){

                // 枚举N个鸡蛋，在M层楼层，所有的可能，如果初始化的值大于计算的值，则计算的值替换原坐标的值
                for(int k=1;k<=m;k++){
                    currentCache[m] = Math.min(currentCache[m], 1 + Math.max(preCache[k-1], currentCache[m-k]));
                }

            }

        }

        return currentCache[floorNum];
    }
```