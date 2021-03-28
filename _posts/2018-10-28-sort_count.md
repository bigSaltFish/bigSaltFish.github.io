---
layout: default
title: 分配排序之"计数排序"
#excerpt: 

---

　　假设，有20个随机整数，取值范围是0到10，需要对其排序。可能第一反应是使用快速排序啊，快排的时间复杂度是O(n*log n)！但是，可不可以比O(n*log n)更快呢？这就是这篇文章要介绍的计数排序(从名字上来看，就是计算数字出现频次的排序方法，非常的见名知意)。

# 计排的实现

　　因为取值范围是0到10（整数的值在0,1,2,3,4,5,6,7,8,9,10里），共有11个数值，所以需要11个坑，我们定义一个长度是12的数组array，每个元素的初始值是0，然后对20个随机整数进行循环，对应的元素值加1。最后，输出array，就是排好序的。

　　比如有20个整数，分别是 9,2,8,5,1,8,6,9,5,8,1,5,8,3,4,7,0,6,2,10

　　初始化数组array

<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>频率</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
        </tr>
    </tbody>
</table>


　　要排序的随机整数，第一个值是9，那数组下标是9的分布值加1，所以array变成
<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>频率</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>1</td>
            <td>0</td>
        </tr>
    </tbody>
</table>

　　第二个值是2，数组下标是2的分布值加1
<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>频率</td>
            <td>0</td>
            <td>0</td>
            <td>1</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>1</td>
            <td>0</td>
        </tr>
    </tbody>
</table>


　　后面的以此类推。最终输出array，元素的值是几，就输出几次。这个数组显然是有序的了。
　　
<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>频率</td>
            <td>1</td>
            <td>2</td>
            <td>2</td>
            <td>1</td>
            <td>1</td>
            <td>3</td>
            <td>2</td>
            <td>1</td>
            <td>4</td>
            <td>2</td>
            <td>1</td>
        </tr>
    </tbody>
</table>


　　代码如下

```python
def sort(array):
    # 1.获取数组内的最大值和最小值，来确定数组的size
	max = array[0]
	min = array[0]
	for i in array:
		if i > max:
			max = i
		if i < min:
			min = i

	# 2.定义定长数组，数组的元素初始值是0（这里偷懒使用了列表生成式）
	sort_array = [0 for i in range(max - min + 1)]

	# 3.计数
	for i in array:
		sort_array[i - min] = sort_array[i - min] + 1

	# 4.获得排序后的数组
	sortArray = [0 for i in array]
	index = 0
	for i, value in enumerate(sort_array):
		for j in range(1, value + 1):
			sortArray[index] = i + min
			index = index + 1
	return sortArray

if __name__ == "__main__":
    array = [9, 2, 8, 5, 1, 8, 6, 9, 5, 8, 1, 5, 8, 3, 4, 7, 0, 6, 2, 10]
    sort_array = sort(array)
    print(sort_array)
```

　　下面是输出结果

```html
[0, 1, 1, 2, 2, 3, 4, 5, 5, 5, 6, 6, 7, 8, 8, 8, 8, 9, 9, 10]

Process finished with exit code 0
```
　　从代码实现来看，也不复杂，甚至很简单。

# 优化版的计排

　　上面的算法，因为有一个双重fou循环，还可以在优化下。

　　假设考试是十分制的，共有20个同学，同学的考试成绩是上面的那20个数（9, 2, 8, 5, 1, 8, 6, 9, 5, 8, 1, 5, 8, 3, 4, 7, 0, 6, 2, 10），考试获得9分的有两个同学，那到底谁是第二、谁是第三呢？

上面的表格，需要升级一下，频率一栏，需要变成“分布值”了，它的值是上一格的值和自身的值之和

<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>分布值</td>
            <td>1</td>
            <td>3(1+2)</td>
            <td>5(3+2)</td>
            <td>6(5+1)</td>
            <td>7(6+1)</td>
            <td>10(7+3)</td>
            <td>12(10+2)</td>
            <td>13(12+1)</td>
            <td>17(13+4)</td>
            <td>19(17+2)</td>
            <td>20(19+1)</td>
        </tr>
    </tbody>
</table>


然后对分布值数组输出，第一个是下标10的，遍历之后分布值表格的下标10的分布值就减1，它是第一名（总人数-分布值+1）
<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>分布值</td>
            <td>1</td>
            <td>3</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>10</td>
            <td>12</td>
            <td>13</td>
            <td>17</td>
            <td>19</td>
            <td>19(20-1)</td>
        </tr>
    </tbody>
</table>
假如获得9分的是张三和李四，遍历张三之后，表格下标9的值减1，它是第二名（2 = 20-19 +1）
<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>分布值</td>
            <td>1</td>
            <td>3</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>10</td>
            <td>12</td>
            <td>13</td>
            <td>17</td>
            <td>18(19-1)</td>
            <td>19</td>
        </tr>
    </tbody>
</table>
李四是第三名(3 = 20 -18 + 1)

<table>
    <thead>
    	<tr>
            <td>数组下标</td>
            <td>0</td>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>8</td>
            <td>9</td>
            <td>10</td>
        </tr>
    </thead>
    <tbody>
    	<tr>
        	<td>分布值</td>
            <td>1</td>
            <td>3</td>
            <td>5</td>
            <td>6</td>
            <td>7</td>
            <td>10</td>
            <td>12</td>
            <td>13</td>
            <td>17</td>
            <td>17(18-1)</td>
            <td>19</td>
        </tr>
    </tbody>
</table>

这样的话，相同分数的同学，就分出分数排名了。

```python
def sort2(array):
    # 1.获取数组内的最大值和最小值，来确定数组的size
    max = array[0]
    min = array[0]
    for i in array:
        if i > max:
            max = i
        if i < min:
            min = i

    # 2.定义定长数组，数组的元素初始值是0（这里偷懒使用了列表生成式）
    countArray = [0 for i in range(max - min + 1)]

    # 3.计数
    for i in array:
        countArray[i - min] = countArray[i - min] + 1

    # 4.计数的数组变形
    sum = 0
    for i,value in enumerate(countArray):
        sum = sum + value
        countArray[i] = sum

    # 5.获得排序后的数组
    sortArray = [0 for i in array]
    for i in array:
        sortArray[countArray[i - min] - 1] = i
        countArray[i - min] = countArray[i - min] - 1

    return sortArray


if __name__ == "__main__":
    array = [9, 2, 8, 5, 1, 8, 6, 9, 5, 8, 1, 5, 8, 3, 4, 7, 0, 6, 2, 10]
    sort_array = sort2(array)
    print(sort_array)
```

# 复杂度

　　假设array元素有N个，取值范围是M。

时间复杂度：3N(步骤1、步骤3、步骤5) + M(步骤4)，去掉系数，时间复杂度是O(N+M)

空间复杂度：只考虑统计数组，那就是M

# 计排的缺陷

1. 数组的元素有小数。比如有一个元素是1.001,这种创建统计数组的话，那代价就太大太大了。
2. 数组内的元素跨度大。假如有十个数，最大是1000，最小是1，使用计排的话，得创建1000个长度的数组，这显然也不合适的。