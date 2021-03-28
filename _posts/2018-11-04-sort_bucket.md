---
layout: default
title: 分配排序之"桶排序"
#excerpt: 

---

　　上一篇说了计数排序，一种稳定的排序，时间复杂度是指数级，但是，计数排序不适用于跨度很大或者浮点数，那么有没有可以处理浮点类型的稳定排序呢？桶排序就是。

# 桶排序的实现

　　首先说一下桶排序的桶是什么概念，这里的“桶”是一个区间范围，里面可以承载一个或多个元素。桶排序的第一步就是确定桶的个数和区间。具体的建立多少个桶、每个桶的区间范围是多少，有不同的方式，我们这里使用桶的数量等于原始数列的元素的数量（为什么等于数列的数量，后面会讲到）。除了最后的一个桶只包含最大值，其他的值分散在其他桶里。

区间跨度 = （最大值-最小值）/ （桶的数量 - 1）

![]({{site.url}}/assets/2018-11-04-sort_bucket/bucket01.png)

第二步是把原始数列的元素放入桶中

![]({{site.url}}/assets/2018-11-04-sort_bucket/bucket02.png)

第三步是桶内的元素进行排序

![]({{site.url}}/assets/2018-11-04-sort_bucket/bucket03.png)

第四步就是遍历所有的桶，输出元素

0.5，0.84，2.18，3.25，4.5

　　上面是思路，具体代码如下：

```python
def sortBucket(array):
    max = array[0]
    min = array[0]
    for i in array:
        if max < i:
            max = i
        if min > i:
            min = i

    # 1.初始化桶
    bucketNumber = len(array)
    bucketArray = []
    for i in range(bucketNumber):
        t = []
        bucketArray.append(t)

    # 2.数列的元素放入桶
    for i in array:
        index = int((i - min) / (max - min) * (bucketNumber - 1))
        bucketArray[index].append(i)

    # 3.桶内的元素排序
    for i in array:
        bucketArray[i].sort()

    # 4.返回排好序的集合
    sortedArray = []
    for bucket in bucketArray:
        for element in bucket:
            sortedArray.append(element)

    return sortedArray

if __name__ == '__main__':
    array = [3, 3, 3, 7, 9, 1, 7, 1, 3, 4]
    bucketArray = sortBucket(array)
    print(bucketArray)
```

输出结果

```html
[1, 1, 3, 3, 3, 3, 4, 7, 7, 9]

Process finished with exit code 0
```
　　这里有一个不是非常明白的地方，就是 第二步中数列元素放入桶，计算index的算法：当前差值/全局差值*数列的长度减一（按照比例定位），在网上也看了其他的文章，有的文章里计算index的算法直接是：当前差值/数列长度。然后，我代入了几个元素，发现，还是前者的算法计算出来的下标更加精确。

# 桶排序的复杂度

　　假设原始数列的元素个数是N，桶的数量的M，平均每个桶内的元素数量的N/M

　　求最值，计算量N

　　初始化桶，计算量M

　　数列的元素放入桶，计算量N

　　每个桶内元素排序，由于使用了O(n log n)算法，计算量是M*(N/M * log N/M)=N*(log N/M)

　　最后返回排好序的集合，计算量是N

　　综上所述，计算量是 3N+M+ N(log N - log M)，去掉系数O(N+M+N(log N - log M))

假如M=N，则时间复杂度O(N+M)，近似O(N)

　　空间复杂度：很明显是原始数组的空间N 加上 桶的空间M，是N+M

# 桶排序的缺陷

  假如元素分布极不均衡，比如全部在最后一个桶内，则此时的时间复杂度就退化到了O(n*log n)，并且空间复杂度浪费了很多，因为创建了很多无用的空桶