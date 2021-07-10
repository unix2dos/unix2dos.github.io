---
title: 常见算法的golang实现-数据结构
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

### 1.1 [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)(简单)

```go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }

    ok := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil
    return ok
}
```

<!-- more -->

+ 循环版本: 头尾四连咬

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

##### 1.2.1 [环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

+ hash

  key 存储节点的引用, 环的入口就是第一个重复的

+ 快慢指针 

  头结点和相遇节点离入口点距离一样
  
  头脑: fast 阶梯判断

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

##### 1.2.2 [环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)(中等)

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



### 1.3 [两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)(中等)

```go
func swapPairs(head *ListNode) *ListNode {
    if head == nil || head.Next == nil{
        return head
    }

    next := head.Next  
    ok := swapPairs(head.Next.Next)
    head.Next.Next = head
    head.Next = ok

    return next
}
```




### 1.4 [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)(简单)

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    if l1 == nil{
        return l2
    }
    if l2 == nil {
        return l1
    }
    
    if l1.Val < l2.Val {
        l1.Next = mergeTwoLists(l1.Next, l2)
        return l1
    }else {
        l2.Next = mergeTwoLists(l1, l2.Next)
        return l2
    }
}
```



### 1.5 [链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)(简单)

+ 先让快指针走k步，然后两个指针同步走，当快指针走到头时，慢指针就是链表倒数第k个节点。

```go

func getKthFromEnd(head *ListNode, k int) *ListNode {
    fast := head
    slow := head
    for k >0 && fast != nil {
        fast = fast.Next
        k--
    }
    for fast != nil {
        fast = fast.Next
        slow = slow.Next
    }
    return slow
}
```



### 1.6 [删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)(简单)

```go
func deleteNode(head *ListNode, val int) *ListNode {
    if head == nil {
        return nil
    }
    head.Next = deleteNode(head.Next, val)
    if head.Val == val {
        return head.Next
    }else{
        return head
    }
}
```



### 1.7 [从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)(简单)

```go
func reversePrint(head *ListNode) []int {
    if head == nil {
        return nil
    }

    res := append(reversePrint(head.Next), head.Val) 
    return res
}
```





# 2. 数组

### 2.1[数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)(简单)

+ 头脑风暴: 临时数组++

  ```go
  func findRepeatNumber(nums []int) int {
  
      arr := make([]int, len(nums), len(nums))
      for i := 0; i < len(nums); i++{
          arr[nums[i]]++   						// 题目说明, <n, 不会越界
          if arr[nums[i]] > 1{
              return nums[i]
          }
      }
  
      return 0
  }
  ```



# 3. 树

### 3.1 [二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)(简单)

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }

    left := maxDepth(root.Left)
    right := maxDepth(root.Right)
    return Max(left,right)+1
}

func Max(a, b int) int{
    if a > b {
        return a
    }
    return b
}
```



### 3.2 [平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)(中等)

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



### 3.3 二叉搜索树最近祖先

+ https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	val := root.Val
    pv := p.Val
    qv := q.Val

    if pv > val && qv > val {
        return lowestCommonAncestor(root.Right, p, q)
    }else if pv < val && qv < val {
        return lowestCommonAncestor(root.Left, p, q)
    }else{
        return root
    }
}
```



### 3.4 [重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)(中等)

+ 画图, len(preorder[:pos])  是左子树的长度

```go
func buildTree(preorder []int, inorder []int) *TreeNode {
    if len(preorder) == 0{
        return nil 
    }

    root := &TreeNode{preorder[0],nil,nil}
    pos := 0
    for ; pos < len(inorder); pos++{
        if inorder[pos] == preorder[0]{
            break
        }
    }

    root.Left = buildTree(preorder[1:len(preorder[:pos])+1],inorder[:pos]) 
    root.Right =  buildTree(preorder[len(preorder[:pos])+1:],inorder[pos+1:]) 
    return root
}
```



### 3.5 [合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)(简单)

```go
func mergeTrees(root1 *TreeNode, root2 *TreeNode) *TreeNode {
    if root1 == nil{
        return root2
    }
    if root2 == nil{
        return root1
    }

    root1.Val = root1.Val + root2.Val
    root1.Left = mergeTrees(root1.Left, root2.Left)
    root1.Right = mergeTrees(root1.Right, root2.Right)
    return root1  
}
```



# 4. 参考资料

+ https://labuladong.gitbook.io/
+ https://lyl0724.github.io/2020/01/25/1/