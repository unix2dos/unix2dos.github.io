---
title: 常见算法的golang实现
tags:
  - golang
  - 算法
categories:
  - 3-计算机系统
  - 算法
abbrlink: 3850b4cc
date: 2019-01-07 00:00:00
---

# 1. 链表

### 1.1 翻转链表

+ https://leetcode-cn.com/problems/reverse-linked-list/

+ 循环版本

```go
func reverseList(head *ListNode) *ListNode {
    var prev,curr *ListNode
    
    curr = head
    for curr != nil {
        tmp := curr.Next
        curr.Next = prev
        prev = curr
        curr = tmp
    }
    return prev
}
```
<!-- more -->

+ 递归版本

```go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil{
        return head
    }
    // 第一步, 第二步, 第三步 疯狂递归
    p := reverseList(head.Next)
  
  // 因为最后一个直接返回, 现在 head 是 倒数第二, p 是倒数第一
    head.Next.Next = head //增加一个指向(原地翻转)
    head.Next = nil// 删除一个指向
    return p//p一直是最后一个, 但是由于递归完毕, 就变成第一个了
}
```



### 1.2 链表有环和入口

+ https://leetcode-cn.com/problems/linked-list-cycle/

+ https://leetcode-cn.com/problems/linked-list-cycle-ii/

+ hash 表判断

  key 存储节点的引用, 环的入口就是第一个重复的

  + 时间复杂度 O(n)

  + 空间复杂度 O(n)

+ 快慢指针

  头结点和相遇节点离入口点距离一样

  + 时间复杂度 O(n)
  
  + 空间复杂度 O(1), 只使用了慢指针和快指针两个结点，所以空间复杂度为 O(1)*O*(1)。

```go
func hasCycle(head *ListNode) bool {
    slow := head
    fast := head
    for (fast != nil && fast.Next != nil){  // 一定要判断 fast
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast{
            return true
        }
    }
    return false
}
```

+ 找入口

```go
func detectCycle(head *ListNode) *ListNode {
    slow := head
    fast := head
    cycle := false
    for (fast != nil && fast.Next != nil){ // 一定要判断 fast
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast{
            cycle = true
            break
        }
    }
    if !cycle{
        return nil
    }

    p1 := head
    p2 := slow // 或者 fast
    for p1 != p2 {
        p1 = p1.Next
        p2 = p2.Next
    }
    return p1
}
```



# 2. 排序

### 2.1 快速排序

+ https://leetcode-cn.com/problems/sort-an-array/

+ 模板

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




### 2.2 归并排序

归并排序利用了分治的思想来对序列进行排序。对一个长为 n 的待排序的序列，我们将其分解成两个长度为 n/2的子序列。每次先递归调用函数使两个子序列有序，然后我们再线性合并两个有序的子序列使整个序列有序。

### 2.3 堆排序

堆排序的思想就是先将待排序的序列建成大根堆，使得每个父节点的元素大于等于它的子节点。此时整个序列最大值即为堆顶元素。

我们将其与末尾元素交换，使末尾元素为最大值，然后再调整堆顶元素使得剩下的 n−1 个元素仍为大根堆，再重复执行以上操作我们即能得到一个有序的序列。

+ 一直大顶堆, 删除顶部, 再大顶堆

# 3. 其他TODO

二分查找

反转字符串 

回文字符串