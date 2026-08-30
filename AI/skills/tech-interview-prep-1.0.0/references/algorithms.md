# 算法面试题库

## 数组与字符串

### 两数之和
**题目**：给定数组和目标值，返回两数之和等于目标值的下标。
**思路**：哈希表存储已遍历元素，O(n) 时间。
**代码**：
```python
def twoSum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        if target - num in seen:
            return [seen[target - num], i]
        seen[num] = i
```

### 滑动窗口最大值
**题目**：给定数组和窗口大小 k，返回每个窗口的最大值。
**思路**：单调递减队列，队首永远是最大值。
**复杂度**：O(n)

### 最长子串无重复字符
**题目**：找不含重复字符的最长子串长度。
**思路**：滑动窗口 + 哈希表记录字符位置。

## 链表

### 反转链表
**迭代法**：
```python
def reverseList(head):
    pre, cur = None, head
    while cur:
        nxt = cur.next
        cur.next = pre
        pre = cur
        cur = nxt
    return pre
```

**递归法**：
```python
def reverseList(head):
    if not head or not head.next:
        return head
    newHead = reverseList(head.next)
    head.next.next = head
    head.next = None
    return newHead
```

### 环形链表检测
**方法**：快慢指针，相遇则有环。
```python
def hasCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

### 合并两个有序链表
**递归实现**：
```python
def mergeTwoLists(l1, l2):
    if not l1: return l2
    if not l2: return l1
    if l1.val < l2.val:
        l1.next = mergeTwoLists(l1.next, l2)
        return l1
    else:
        l2.next = mergeTwoLists(l1, l2.next)
        return l2
```

## 二叉树

### 前中后序遍历
**递归版**：最简单，但面试官可能要求迭代。
**迭代版**：用栈模拟递归过程。

### 层序遍历
**方法**：BFS，用队列。
```python
def levelOrder(root):
    if not root: return []
    result, queue = [], [root]
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.pop(0)
            level.append(node.val)
            if node.left: queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result
```

### 最近公共祖先 LCA
**思路**：递归，左右子树都找到则当前节点是 LCA。

## 动态规划

### 斐波那契数列
**基础版**：dp[i] = dp[i-1] + dp[i-2]
**优化版**：只存前两个数，空间 O(1)

### 爬楼梯
**题目**：每次爬 1 或 2 步，n 阶有多少种方法？
**答案**：就是斐波那契，f(n) = f(n-1) + f(n-2)

### 最长递增子序列 LIS
**O(n²) 方法**：dp[i] = 以 i 结尾的 LIS 长度
**O(n log n) 方法**：贪心 + 二分查找

### 编辑距离
**题目**：word1 变成 word2 最少操作次数。
**状态转移**：
- dp[i][j] = word1[:i] 和 word2[:j] 的编辑距离
- 如果 word1[i-1] == word2[j-1]：dp[i][j] = dp[i-1][j-1]
- 否则：dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1

### 背包问题
**01背包**：每个物品只能选一次
**完全背包**：每个物品可选多次
**状态转移**：
- 01背包：dp[i][w] = max(dp[i-1][w], dp[i-1][w-wi] + vi)

## 排序算法

| 算法 | 平均 | 最坏 | 空间 | 稳定 |
|------|------|------|------|------|
| 冒泡 | O(n²) | O(n²) | O(1) | 是 |
| 选择 | O(n²) | O(n²) | O(1) | 否 |
| 插入 | O(n²) | O(n²) | O(1) | 是 |
| 快排 | O(n log n) | O(n²) | O(log n) | 否 |
| 归并 | O(n log n) | O(n log n) | O(n) | 是 |
| 堆排 | O(n log n) | O(n log n) | O(1) | 否 |

### 快速排序
```python
def quickSort(arr, lo, hi):
    if lo >= hi: return
    pivot = partition(arr, lo, hi)
    quickSort(arr, lo, pivot - 1)
    quickSort(arr, pivot + 1, hi)

def partition(arr, lo, hi):
    pivot = arr[hi]
    i = lo
    for j in range(lo, hi):
        if arr[j] < pivot:
            arr[i], arr[j] = arr[j], arr[i]
            i += 1
    arr[i], arr[hi] = arr[hi], arr[i]
    return i
```

## 图算法

### BFS 模板
```python
from collections import deque
def bfs(graph, start):
    visited = set([start])
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

### DFS 模板
```python
def dfs(graph, node, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```

### 拓扑排序
**应用**：课程安排、编译依赖
**方法**：Kahn 算法（入度为 0 入队）

### 最短路径
- **Dijkstra**：正权图，单源最短
- **Bellman-Ford**：可以有负权
- **Floyd-Warshall**：多源最短
