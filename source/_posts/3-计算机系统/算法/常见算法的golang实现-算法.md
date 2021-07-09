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

+ 为什么不小顶堆

  因为建好的小顶堆也不是排序好的

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



# 3. 其他

### 3.1 [LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)(中等)

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



### 3.2 温度

+ https://leetcode-cn.com/problems/daily-temperatures/



# 4. 脑力风暴

### 4.1 递归

+ 第一步考虑出口

+ 第二步, 为了目的, 我应该完成怎么样的返回值

+ 第三步, 例如考虑倒数第二层或简单理解的层, 只考虑当前层的逻辑

  

# 5. TODO

反转字符串 

回文字符串

最长公用子串

KMP

脑力风暴加强



# 6. 参考资料

+ https://labuladong.gitbook.io/
+ https://lyl0724.github.io/2020/01/25/1/