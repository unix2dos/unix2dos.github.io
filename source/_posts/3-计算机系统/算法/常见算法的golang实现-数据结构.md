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

### + [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)(简单🔥)

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



### + [环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)(简单)

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

### + [环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)(中等)

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



### + [两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)(中等)

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




### + [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)(简单)[合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)（🔥）

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    if l1 == nil {
        return l2
    }
    if l2 == nil {
        return l1
    }

    if l1.Val < l2.Val{
        l1.Next = mergeTwoLists(l1.Next, l2)
        return l1
    }else{
        l2.Next = mergeTwoLists(l2.Next,l1)
        return l2
    }
}
```



### + [链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)(简单)

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



### + [删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)(简单)

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



### + [从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)(简单)

```go
func reversePrint(head *ListNode) []int {
    if head == nil {
        return nil
    }

    res := append(reversePrint(head.Next), head.Val) 
    return res
}
```



### + [两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)(简单) [相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

- 程序尽量满足 O(*n*) 时间复杂度，且仅用 O(*1*) 内存。
- 两个指针( 两个链表长度分别为L1+C、L2+C， C为公共部分的长度，按照楼主的做法： 第一个人走了L1+C步后，回到第二个人起点走L2步；第2个人走了L2+C步后，回到第一个人起点走L1步。 当两个人走的步数都为L1+L2+C时就两个家伙就相爱了)

```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    pos1 := headA
    pos2 := headB
    for pos1 != pos2 {
        if pos1 != nil{
            pos1 = pos1.Next
        }else{
            pos1 = headB
        }

        if pos2 != nil {
            pos2 = pos2.Next
        }else{
            pos2 = headA
        }
    }
    return pos2
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

### + [二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)(简单) [二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

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



### + [平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)(中等) [平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)(🔥)

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



### + [重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)(中等)

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



### + [合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)(简单)

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

### + [相同的树](https://leetcode-cn.com/problems/same-tree/)(简单)

```go
func isSameTree(p *TreeNode, q *TreeNode) bool {
    if p == nil && q == nil {
        return true
    }
    if p == nil || q == nil {
        return false
    }

    return p.Val == q.Val && isSameTree(p.Left, q.Left) && isSameTree(p.Right, q.Right)
}
```



### + [另一个树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree/)(简单)

```go
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    if root == nil || subRoot == nil {
        return false
    }

    if isSameTree(root, subRoot) {
        return true
    }

    return isSubtree(root.Left, subRoot) || isSubtree(root.Right, subRoot)
}


func isSameTree(p *TreeNode, q *TreeNode) bool {
  if p == nil && q == nil {
        return true
    }
    if p == nil || q == nil {
        return false
    }

    return p.Val == q.Val && isSameTree(p.Left, q.Left) && isSameTree(p.Right, q.Right)
}
```



### + [树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)(中等)

```go
func isSubStructure(A *TreeNode, B *TreeNode) bool {
    if A == nil || B == nil {
        return false
    }

    return isContain(A, B) || isSubStructure(A.Left, B) || isSubStructure(A.Right, B)
    
}


// A,B根节点相同，B是不是A的子结构
func isContain(A *TreeNode, B *TreeNode) bool {
    if B == nil {
        return true
    }
    if A == nil {
        return false
    }
    if A.Val != B.Val {
        return false
    }
    return isContain(A.Left, B.Left) && isContain(A.Right, B.Right)
}
```



### + [单值二叉树](https://leetcode-cn.com/problems/univalued-binary-tree/)(简单)

```go
func isUnivalTree(root *TreeNode) bool {
    if root == nil {
        return true
    }
    if root.Left != nil && root.Val != root.Left.Val {
        return false
    }
    if root.Right != nil && root.Val != root.Right.Val {
        return false
    }
    return isUnivalTree(root.Left) && isUnivalTree(root.Right)
}
```



### + [翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)(简单) [二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)(🔥)

+ brew作者失败的题

```go
func mirrorTree(root *TreeNode) *TreeNode {
    if root == nil{
        return nil
    }

    left := mirrorTree(root.Left)
    right := mirrorTree(root.Right)
    root.Left = right
    root.Right = left
    return root
}
```



### + [对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)(简单)

```go
func isSymmetric(root *TreeNode) bool {
    if root == nil {
        return true
    }
    return isMirror(root.Left, root.Right)
}


func isMirror(l, r *TreeNode) bool {
    if l == nil && r == nil {
        return true
    }
    if l == nil || r == nil{
        return false
    }

   return l.Val == r.Val && isMirror(l.Right, r.Left) && isMirror(l.Left,r.Right)
}
```



### + [二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)(简单) 

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



### + [二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/) (中等)

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    if p == root || q == root {
        return root
    }

    l := lowestCommonAncestor(root.Left, p, q)
    r := lowestCommonAncestor(root.Right,p, q)
    if l != nil && r != nil {// 左右各一个
        return root
    }
    if l != nil {// 两都在左边
        return l
    }
    if r != nil{ // 两都在右边
        return r
    }

    return nil
}
```



### + [二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)(简单)

```go
var arr[]int

func kthLargest(root *TreeNode, k int) int {
    if root == nil {
        return 0
    }
    arr = []int{}
    dfs(root)
    return arr[k-1]
}

func dfs(root *TreeNode) {
    if root == nil  {
        return 
    }
    dfs(root.Right)
    arr = append(arr, root.Val)
    dfs(root.Left)
}
```

### + [二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)(中等) TODO

### + [二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)(中等) TODO

### + [二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)(中等) TODO

# 4. 字符串

### + [ 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)(简单)

```go
func reverseWords(s string) string {

    s = strings.TrimSpace(s)

    j := len(s)-1
    i := j
    res := []string{}
    for i >= 0 {
        for i >= 0 &&  s[i] != ' '{
            i--
        }
        res = append(res, s[i+1:j+1])
        for i >= 0 &&  s[i] == ' '{
            i--
        }
        j = i

    }

    return strings.Join(res," ")
}
```

### + [罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer/)(简单)

```go
func romanToInt(s string) int {
  res := map[string]int{
        "I": 1,
        "V": 5,
        "X": 10,
        "L": 50,
        "C": 100,
        "D": 500,
        "M": 1000,
    }
    sum := 0
    for i := 0; i < len(s); i++ {
        val := res[string(s[i])]
        if i < len(s)-1 && val < res[string(s[i+1])] {
            sum -= val
        } else {
            sum += val
        }
    }
    return sum
}
```



# 5. 参考资料

+ https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/solution/yi-pian-wen-zhang-dai-ni-chi-tou-dui-che-uhgs/