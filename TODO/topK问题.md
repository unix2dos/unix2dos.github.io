---
title: "topK问题"
date: 2020-08-05 00:00:00
tags:
- 算法
---

# 1. 描述

TopK Elements 问题用于找出一组数中最大的或最小的 K 个的数。此外还有一种叫 Kth Element 问题，用于找出一组数中第 K 大的数。

<!-- more -->

### 1.1 最大

+ https://leetcode-cn.com/problems/kth-largest-element-in-an-array/

  + 堆
  
  ```go

  ```
  
  + 快速选择
  
  ```go
  
  ```
  
  + 比较
  
    看起来分治法的快速选择算法的时间、空间复杂度都优于使用堆的方法，但是要注意到快速选择算法的几点局限性：
  
    + 算法需要修改原数组，如果原数组不能修改的话，还需要拷贝一份数组，空间复杂度就上去了。
  
    + 算法需要保存所有的数据。如果把数据看成输入流的话，使用堆的方法是来一个处理一个，不需要保存数据，只需要保存 k 个元素的最大堆。而快速选择的方法需要先保存下来所有的数据，再运行算法。当数据量非常大的时候，甚至内存都放不下的时候，就麻烦了。所以当数据量大的时候还是用基于堆的方法比较好。
  
    


### 1.2 最小

+ https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof

  ```go
  1
  ```

### 1.3 最小

+ https://leetcode-cn.com/problems/smallest-k-lcci/

  ```go
  1
  ```



# 2. TOP最小的解法

### 2.1 排序

对原数组从小到大排序后取出前 k 个数即可。

```cpp
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        vector<int> vec(k, 0);
        sort(arr.begin(), arr.end());
        for (int i = 0; i < k; ++i) vec[i] = arr[i];
        return vec;
    }
};
```

+ 时间复杂度：O(nlogn)

  其中 nn 是数组 arr 的长度。算法的时间复杂度即排序的时间复杂度。

+ 空间复杂度：O(logn)

  排序所需额外的空间复杂度为 O(logn)。



### 2.2 堆

我们用一个大根堆实时维护数组的前 k 小值。首先将前 k 个数插入大根堆中，随后从第 k+1 个数开始遍历，如果当前遍历到的数比大根堆的堆顶的数要小，就把堆顶的数弹出，再插入当前遍历到的数。最后将大根堆里的数存入数组返回即可。(C++优先队列即是大根堆)

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        vector<int>vec(k, 0);
        if (k == 0) return vec; // 排除 0 的情况
      
        priority_queue<int>Q;
        for (int i = 0; i < k; ++i) Q.push(arr[i]);
      
        for (int i = k; i < (int)arr.size(); ++i) {
            if (Q.top() > arr[i]) {
                Q.pop();
                Q.push(arr[i]);
            }
        }
      
        for (int i = 0; i < k; ++i) {
            vec[i] = Q.top();	
            Q.pop();
        }
      
        return vec;
    }
};
```



+ 时间复杂度：O(nlogk)

  其中 n 是数组 arr 的长度。由于大根堆实时维护前 k 小值，所以插入删除都是 O(logk) 的时间复杂度，最坏情况下数组里 n 个数都会插入，所以一共需要 O(nlogk) 的时间复杂度。

+ 空间复杂度：O(k)

  因为大根堆里最多 k 个数。



### 2.3 快排思想

我们可以借鉴快速排序的思想。我们知道快排的划分函数每次执行完后都能将数组分成两个部分，小于等于分界值 pivot 的元素的都会被放到数组的左边，大于的都会被放到数组的右边，然后返回分界值的下标。与快速排序不同的是，快速排序会根据分界值的下标递归处理划分的两侧，而这里我们只处理划分的一边。



我们定义函数 randomized_selected(arr, l, r, k) 表示划分数组 arr 的 [l,r] 部分，使前 k 小的数在数组的左侧，在函数里我们调用快排的划分函数，假设划分函数返回的下标是 pos（表示分界值 pivot 最终在数组中的位置），即 pivot 是数组中第 pos - l + 1 小的数，那么一共会有三种情况：

+ 如果 pos - l + 1 == k，表示 pivot 就是第 k 小的数，直接返回即可；

+ 如果 pos - l + 1 < k，表示第 k 小的数在 pivot 的右侧，因此递归调用 randomized_selected(arr, pos + 1, r, k - (pos - l + 1))；

+ 如果 pos - l + 1 > k，表示第 k 小的数在 pivot 的左侧，递归调用 randomized_selected(arr, l, pos - 1, k)。

函数递归入口为 randomized_selected(arr, 0, arr.length - 1, k)。在函数返回后，将前 k 个数放入答案数组返回即可。

```cpp
class Solution {
    int partition(vector<int>& nums, int l, int r) {
        int pivot = nums[r];
        int i = l - 1;
        for (int j = l; j <= r - 1; ++j) {
            if (nums[j] <= pivot) {
                i = i + 1;
                swap(nums[i], nums[j]);
            }
        }
        swap(nums[i + 1], nums[r]);
        return i + 1;
    }
  
    // 基于随机的划分
    int randomized_partition(vector<int>& nums, int l, int r) {
        int i = rand() % (r - l + 1) + l;
        swap(nums[r], nums[i]);
        return partition(nums, l, r);
    }
  
  
    void randomized_selected(vector<int>& arr, int l, int r, int k) {
        if (l >= r) return;
        int pos = randomized_partition(arr, l, r);
        int num = pos - l + 1;
        if (k == num) return;
        else if (k < num) randomized_selected(arr, l, pos - 1, k);
        else randomized_selected(arr, pos + 1, r, k - num);   
    }
  
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        srand((unsigned)time(NULL));
        randomized_selected(arr, 0, (int)arr.size() - 1, k);
      
        vector<int>vec;
        for (int i = 0; i < k; ++i) vec.push_back(arr[i]);
        return vec;
    }
};
```



+ 时间复杂度：期望为 O(n) ，最坏情况下的时间复杂度为 O(n^2)。情况最差时，每次的划分点都是最大值或最小值，一共需要划分 n−1 次，而一次划分需要线性的时间复杂度，所以最坏情况下时间复杂度为 O(n^2)。

+ 空间复杂度：期望为 O(logn)，递归调用的期望深度为 O(logn)，每层需要的空间为 O(1)，只有常数个变量。

  最坏情况下的空间复杂度为 O(n)。最坏情况下需要划分 n 次，即 randomized_selected 函数递归调用最深 n - 1 层，而每层由于需要 O(1) 的空间，所以一共需要 O(n) 的空间复杂度。



# 3. 头脑风暴

最大的 TOPK, 就是小顶堆

最小的 TOPK, 就是大顶堆



# 4. 参考资料

+ https://xiaozhuanlan.com/topic/4176082593
+ https://zhuanlan.51cto.com/art/201809/584259.htm
+ https://github.com/sisterAn/JavaScript-Algorithms/issues/73
+ https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution/shu-zu-zhong-de-di-kge-zui-da-yuan-su-by-leetcode-/
+ https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/solution/zui-xiao-de-kge-shu-by-leetcode-solution/
+ https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/solution/tu-jie-top-k-wen-ti-de-liang-chong-jie-fa-you-lie-/