---
layout: default
title: 分配排序之"基数排序"
#excerpt: 

---

　　上一篇说了桶排序，这次说一下桶排序的扩展-基数排序。

# 基本概念

　　要说基数排序的话，那就必须得说一下两个概念。“基”和“桶”

　　基：基数排序里的“基”是什么意思呢？基的英文是radix，直接翻译是进制的意思，在基数排序里，指的是数字的位，比如数字123，这里1是百位的基，2是十位的基，3是个位的基

　　桶：桶就是上一篇桶排序中的那个桶，表示一个区间内的值。

# 基数排序的实现

　　那么基数排序是如何进行排序的呢？比如，有一组数字[9, 11, 31, 27, 50]，使用基数排序的话，它先对个位数 [9,1,1,7,0]进行排序，再对十位进行排序，对个位和十位都排好序了，那整组数字就是排好序的了。

1. 以个位数为依据

   根据个位定位丢进哪个桶内

   ![根据个位定位丢进哪个桶内]({{site.url}}/assets/2018-11-10-sort_radix/radix1.jpg)

   遍历array，元素放入对应的桶内

   ![丢进桶里]({{site.url}}/assets/2018-11-10-sort_radix/radix1_sorted.jpg)

   遍历桶，输出元素到array

   ![输出到array]({{site.url}}/assets/2018-11-10-sort_radix/outBucket.jpg)

2. 以十位为依据

   根据十位定位应该丢进哪个桶

![根据个位定位丢进哪个桶内]({{site.url}}/assets/2018-11-10-sort_radix/radix10.jpg)

​      遍历桶，输出排好序的元素

![十位排序]({{site.url}}/assets/2018-11-10-sort_radix/radix10_sorted.jpg)



伪代码如下：

for 每个基:

​    第一步：遍历要排序的集合arr，放入对应的桶bucket

​    第二步：把桶里的元素放回arr



程序代码如下：

```python
def radixSort(array):
    # 1.获取数组内的最大值、初始化基
    maxNumber = max(array)
    radix = 1

    # 2.循环排序每个基
    sortedArray = []
    while maxNumber // radix > 0:
        # 1.初始化桶数组，桶数组里有十个桶，每个桶是一个集合
        bucketArray = [[] for i in range(10)]

        # 2.所有元素丢进对应的桶
        for i in array:
            # 元素属于哪个桶
            bucketIndex = i / radix % 10
            bucket = bucketArray[bucketIndex]
            bucket.append(i)

        # 3.排好radix位的有序集合，返回给while外定义的集合来接收
        sortedArray = []
        for bucket in bucketArray:
            for i in bucket:
                sortedArray.append(i)

        # 4.下一次遍历 更高一位
        radix = radix * 10

    return sortedArray

if __name__ == '__main__':
	array = [9, 11, 31, 27, 50]
	sortedAray = radixSort(array)
	print(sortedAray)
```
　　输出结果：

```html
[9, 11, 27, 31, 50]

Process finished with exit code 0
```



# 复杂度

*时间复杂度* ：O(10 + r * (10 + n + n) )，其中，r是radix，如果最大数是1000，那n就是4

　　前面的10影响很小，可以去掉，复杂度O(r * (10 + n + n) )

　　r * (10 + n + n)内的10，在n很大的时候，也可以直接忽略，复杂度O(r * (n + n) )=O(2*rn)

　　同样当n很大的时候，n和2*rn是一个意义的，去掉系数，时间复杂度是O(n)

所以，可以近似的认为，基数排序的时间复杂度是O(n)

*空间复杂度*：O(n + rd)

　　传入的集合中元素个数为n，r是位数(桶的个数)，d是每一位的个数(桶内的元素个数)



# 基排的缺点

1. 基数排序不能处理负数（当然数组内全部加上一个正常数，使数组内全部是正数，基排好后，再减去这个正常数。这样也可以的，但是，代价有点大）

2. 适合数据比较集中的场景，如果数据跨度很大，会造成取“基”的成本很高。