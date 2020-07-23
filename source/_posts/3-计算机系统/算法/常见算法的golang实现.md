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


+ 递归版本

```go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil{
        return head
    }
    p := reverseList(head.Next)
    head.Next.Next = head 
    head.Next = nil
    return p
}
```

<!-- more -->

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



### 1.2 链表有环和入口

+ https://leetcode-cn.com/problems/linked-list-cycle/

+ https://leetcode-cn.com/problems/linked-list-cycle-ii/

+ hash 表判断

  key 存储节点的引用, 环的入口就是第一个重复的

+ 快慢指针

  头结点和相遇节点离入口点距离一样


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



### 1.3 两两交换链表中的节点

+ https://leetcode-cn.com/problems/swap-nodes-in-pairs/

  ```go
  func swapPairs(head *ListNode) *ListNode {
      if head == nil || head.Next == nil{
          return head
      }
  
      //这一层3个节点, head, next, swapPairs()
      //换成 next, head, swapPairs()
  
      next := head.Next
      head.Next = swapPairs(next.Next)
      next.Next = head
  
      return next
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



# 3. 树

### 3.1 二叉树最大深度

+ https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/

  ```go
  func maxDepth(root *TreeNode) int {
      if root == nil{
          return 0
      }
      left := maxDepth(root.Left)
      right := maxDepth(root.Right)
      return max(left,right)+1
  }
  
  func max(a,b int) int {
      if a > b {
          return a
      }
      return b
  }
  ```

  

### 3.2 平衡二叉树

+ https://leetcode-cn.com/problems/balanced-binary-tree/

```go
func isBalanced(root *TreeNode) bool {
    if root == nil{
        return true
    }
    if !isBalanced(root.Left) || !isBalanced(root.Right){
        return false
    }
    if abs(maxDepth(root.Left)-maxDepth(root.Right)) > 1 {
        return false
    }
    return true
}

func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return max(maxDepth(root.Left),maxDepth(root.Right)) + 1
}

func max(a,b int) int {
    if a > b {
        return a
    }
    return b
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a 
}
```



# 9. 脑力风暴

### 9.1 递归

+ 第一步考虑出口

+ 第二步, 为了目的, 我应该完成怎么样的返回值

+ 第三步, 建议考虑倒数第二层, 只考虑当前层的逻辑

  

# 10. 其他TODO

二分查找

反转字符串 

回文字符串