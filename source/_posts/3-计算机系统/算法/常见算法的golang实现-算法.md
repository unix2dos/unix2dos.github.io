---
title: 常见算法的golang实现-算法
tags:
  - golang
  - 算法
date: 2019-01-07 00:00:01
---

# 1. 查找

### 1.1 [二分查找](https://leetcode-cn.com/problems/binary-search/)(简单)

```go
func search(nums []int, target int) int {    
    left := 0 
    right := len(nums)-1
  
    for left <= right {
        mid := (left + right)/2
        if nums[mid] > target{
            right = mid - 1 
        }else if nums[mid] < target{
          left = mid  + 1
        }else {
         return mid
        }
    }
    return -1
}
```

<!-- more -->

# 2. 排序

### 2.1 快速排序

+ https://leetcode-cn.com/problems/sort-an-array/

  ```go
  func parition(nums []int, left, right int)  int{
  
      tmp := nums[left]
  
      for (left < right){
          for (left < right && nums[right] >= tmp) {
              right--
          }
          nums[left],nums[right] = nums[right],nums[left]
          
          for (left < right && nums[left] <= tmp) {
              left++
          }
          nums[left],nums[right] = nums[right],nums[left]
      }
  
      return left
  }
  
  func quickSort(nums []int, left, right int) {
      if left >= right{
          return
      }
      mid := parition(nums, left, right)
      quickSort(nums, left, mid-1)
      quickSort(nums, mid+1, right)
  }
  
  func sortArray(nums []int) []int {
      quickSort(nums,0,len(nums)-1)
      return nums
  }
  ```



### 2.2 归并排序TODO

归并排序利用了分治的思想来对序列进行排序。对一个长为 n 的待排序的序列，我们将其分解成两个长度为 n/2的子序列。每次先递归调用函数使两个子序列有序，然后我们再线性合并两个有序的子序列使整个序列有序。



### 2.3 堆排序

+ https://www.bilibili.com/video/BV1Eb41147dK

1. 堆排序的思想就是先将待排序的序列建成大根堆，使得每个父节点的元素大于等于它的子节点。此时整个序列最大值即为堆顶元素。

2. 我们将其与末尾元素交换，使末尾元素为最大值，然后再调整堆顶元素使得剩下的 n−1 个元素仍为大根堆
3. 再重复 2 的操作我们即能得到一个有序的序列。

4. 为什么不小顶堆? 因为建好的小顶堆也不是排序好的

```go
package main

import "fmt"

func heapify(arr []int, i int, length int) {
	if i >= length {
		return
	}
	// i和自己左孩子, 右孩子去比较, 如果大, 就交换, 保持顶部最大
	max := i
	lChild := i*2 + 1
	rChild := i*2 + 2
	if lChild <= length && arr[lChild] > arr[max] {
		max = lChild
	}
	if rChild <= length && arr[rChild] > arr[max] {
		max = rChild
	}
	if max != i {
		// 交换
		arr[i], arr[max] = arr[max], arr[i]
		heapify(arr, max, length)
	}
}

func main() {
	arr := []int{3, 5, 3, 0, 8, 6, 1, 5, 8, 6, 2, 4, 9, 4, 7, 0, 1, 8, 9, 7, 3, 1, 2, 5, 9, 7, 4, 0, 2, 6}
	fmt.Println(arr)

	// 构建大顶堆
	length := len(arr) - 1
	for i := length / 2; i >= 0; i-- {
		heapify(arr, i, length)
	}
	fmt.Println(arr)

	// 排序
	for i := length; i >= 1; i-- {
		arr[i], arr[0] = arr[0], arr[i]
		length-- // 切腿
		heapify(arr, 0, length)
	}
	fmt.Println(arr)
}

```





# 3  TopK

### 3.1 二叉堆

把二叉堆看作一个数组，也可以被看作一个完全二叉树，通俗来讲二叉堆其实就是利用完全二叉树的结构来维护的一维数组，按照堆的特点可以把堆分为大顶堆和小顶堆。

+ 大顶堆：每个结点的值都大于或等于其左右孩子结点的值  (升序, 因为顶和末尾交换, 末尾就最大了啊)

+ 小顶堆：每个结点的值都小于或等于其左右孩子结点的值  (降序)

因为不是完全二叉树, 就不好用数组表示了

> 坐标: (2i + 1, 2i + 2)

### 3.2 二叉堆和二叉查找树的区别

1. 逻辑结构

+ 二叉堆是一种完全二叉树，分大根堆、小根堆两种，子结点总是大于或小于父结点。

+ 二叉查找树是一种特殊的二叉树，左儿子结点小于父结点，右儿子结点大于父结点。所谓的AVL树、红黑树等复杂一些的树状数据结构，很多都是二叉查找树优化得到的。

2. 存储结构

+ 二叉堆是一种完全二叉树，所以可以按照 id 编号，通过数组存取

+ 二叉树是链接存储的，就像单链表是指针连接起来的，二叉查找树自然也是链接存储。

3. 时空性能

+ 二叉堆每次调整都是O(logN)，只考虑一个儿子节点和父结点的交换与否。建堆O(N)，一旦堆建起来就很方便很灵活，用堆进行排序也只需要O(NlogN)，堆排序属于插入排序，在频繁增删元素的情况下，维护一个堆往往很划算。
+ 二叉查找树好似顺序存储结构的二分查找，只不过有可能很不平衡形成斜树，造成线性的搜索复杂度，故而需要AVL树、红黑树这些更“平衡”的树，稍有额外调整的损耗，却保证了对数的搜索时间。

4. 实用功能

+ 堆的话一般用于堆排序、构建优先队列这样的情形；
+ 而二叉查找树主要就是搜索。



### 3.3 小顶堆, 大顶堆

+ top100热词, 肯定用小顶堆, 就和堆顶比较就好, 大于就换顶

+ 构建大顶堆

  ```go
  func main() {
  	arr := []int{4, 5, 3, 7, 2, 1, 2}
  	fmt.Println(arr)
  	buildHeap(arr)
  	fmt.Println(arr)
  }
  
  func buildHeap(arr []int) {
  	for i := (len(arr) - 1) / 2; i >= 0; i-- {
  		heapify(arr, i, len(arr))
  	}
  }
  
  func heapify(arr []int, index int, length int) {
  	l := index*2 + 1
  	r := index*2 + 2
  
  	temp := index
  	if l < length && arr[l] > arr[temp] {
  		temp = l
  	}
  	if r < length && arr[r] > arr[temp] {
  		temp = r
  	}
  
  	if temp != index {
  		arr[temp], arr[index] = arr[index], arr[temp]
  		heapify(arr, temp, length)
  	}
  }
  
  /*
  [4 5 3 7 2 1 2]
  [7 5 3 4 2 1 2]
  */
  ```

  

### 3.4 [最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)(简单🔥) [最小K个数](https://leetcode-cn.com/problems/smallest-k-lcci/)(中等)

+ 为什么不使用小顶堆:

  大顶堆维护了最小的几个, 顶最大, k后面的值可以方便和顶相比

  如果使用小顶堆也可以, 需要全部入堆

```go
func getLeastNumbers(arr []int, k int) []int {
  
	buildHeap(arr, k)
	
  for i := k; i < len(arr); i++ {
		if arr[i] < arr[0] {
			arr[i], arr[0] = arr[0], arr[i]
			heapify(arr, 0, k)
		}
	}
	res := []int{}
	for i := 0; i < k; i++ {
		res = append(res, arr[i])
	}
	return res
  
}

func buildHeap(arr []int, size int) {
	for i := size / 2; i >= 0; i-- {
		heapify(arr, i, size)
	}
}

func heapify(arr []int, index int, length int) {
	l := index*2 + 1
	r := index*2 + 2

	temp := index
	if l < length && arr[l] > arr[temp] {
		temp = l
	}
	if r < length && arr[r] > arr[temp] {
		temp = r
	}

	if temp != index {
		arr[temp], arr[index] = arr[index], arr[temp]
		heapify(arr, temp, length)
	}
}
```



### 3.5 [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)(中等🔥)

使用的大顶堆, 需要全入堆

```go
func findKthLargest(nums []int, k int) int {
	buildHeap(nums, len(nums))

	length := len(nums) - 1
	for i := 0; i < k; i++ {
		nums[0], nums[length] = nums[length], nums[0]
		length--
		heapify(nums, 0, length)
	}
	return nums[len(nums)-k]
}

func buildHeap(arr []int, size int) {
	for i := size / 2; i >= 0; i-- {
		heapify(arr, i, size)
	}
}

func heapify(arr []int, index int, length int) {
	l := index*2 + 1
	r := index*2 + 2

	temp := index
	if l < length && arr[l] > arr[temp] {
		temp = l
	}
	if r < length && arr[r] > arr[temp] {
		temp = r
	}

	if temp != index {
		arr[temp], arr[index] = arr[index], arr[temp]
		heapify(arr, temp, length)
	}
}
```



### 3.6 [前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)(中等TODO)

```go
```



### 

# 4. 其他

### 4.1 [LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)(中等)

```go
type LRUCache struct {
    Capacity int
    Map map[int]int
    List []int
}


func Constructor(capacity int) LRUCache {
  return LRUCache{
        Capacity : capacity,
        Map : make(map[int]int, 0),
        List : make([]int, 0),
    }
}

func DelKey(a []int, key int) []int{
    for i := 0; i < len(a); i++ {
		if a[i] == key {
            return append(a[:i], a[i+1:]...)
		}
	}
	return a[1:len(a)]
}

func (this *LRUCache) Get(key int) int {
    if val, ok := this.Map[key]; ok{
        this.List = DelKey(this.List, key)
        this.List = append(this.List, key)
        return val
    }
    return -1
}


func (this *LRUCache) Put(key int, value int)  {

    if _, ok := this.Map[key]; ok{
        this.Map[key] = value
        this.List = DelKey(this.List, key)
        this.List = append(this.List, key)
        return 
    }

    if len(this.Map) >= this.Capacity{
        delKey := this.List[0]      
        this.List = DelKey(this.List, key)
        delete(this.Map, delKey)
    }

    this.Map[key] = value
    this.List = append(this.List, key)
}
```



### 4.2 [ 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```go
func spiralOrder(matrix [][]int) []int {
    if len(matrix) == 0 || len(matrix[0]) == 0{
        return []int{}
    }

    res := []int{}
    l := 0
    r := len(matrix[0]) - 1
    t := 0
    b :=  len(matrix) - 1

    for {
        // 左到右
        for i := l ; i <= r; i++ {
            res = append(res, matrix[t][i])
        }
        t++
        if t > b {
           break 
        }

        // 右到下
        for i := t; i <= b; i++ {
            res = append(res, matrix[i][r])
        }

        r--
        if l > r {
            break
        }

        // 下到左
        for i := r; i >= l; i-- {
            res = append(res, matrix[b][i])
        }
        b--
        if t > b {
           break 
        }


        // 左到上
        for i := b; i >= t; i-- {
            res = append(res, matrix[i][l])
        }
        l++
        if l > r {
            break
        }
    }

    return res
}
```



### 4.3 温度(TODO)

+ https://leetcode-cn.com/problems/daily-temperatures/





# 9. 脑力风暴

### 9.1 递归

适合链表, 和树结构

首先考虑出口, 没有出口死循环了

本次递归做什么, 向上一层返回什么(把自己放在递归中间位置考虑)

先考虑最小递归层, 小的都整不对,别提大的了, 然后可以多用一些临时变量



# 10. 参考资料

+ https://lyl0724.github.io/2020/01/25/1/