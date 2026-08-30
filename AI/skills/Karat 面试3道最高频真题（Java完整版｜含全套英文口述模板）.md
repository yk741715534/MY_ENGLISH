# Karat 面试3道最高频真题（Java完整版｜含全套英文口述模板）

三道全覆盖 Karat 三大高频题型：**哈希表、数组遍历、滑动窗口**，完全贴合官方出题风格，每道题包含：英文读题、思路口述、复杂度、完整Java代码、测试用例口述，100%适配Karat打分Rubric。

## Question 1：Two Sum 两数之和（Karat 必考｜出场率最高）

### 1\. English Problem

Given an integer array nums and an integer target, return the indices of the two numbers that add up to target\. Each input has exactly one solution, and you may not use the same element twice\.

### 2\. Full English Speaking Script（全程口述）

**Restate**: Let me confirm the problem\. I need to find two distinct elements in the array whose sum equals the target, and return their indices\. There is exactly one valid answer\.

**Edge Case**: First, I handle edge cases\. If the array is null or the length is less than 2, return an empty array\.

**Solution Idea**: The brute force approach uses two loops with O\(n²\) time, which is slow\. I will use a HashMap for optimization\. I store each number and its index during traversal\. For each number, I calculate the complement which equals target minus current value\. If the complement exists in the map, we found the result\. Otherwise, put the current number and index into the map\.

**Complexity**: Time complexity is O\(n\), space complexity is O\(n\)\.

**Test Case**: For example, nums = \[2,7,11,15\], target = 9\. The first number is 2, complement 7 is not found, so we store it\. The second number is 7, complement 2 exists in the map\. We return index 0 and 1\. The result is correct\.

### 3\. Java Full Code（可直接在Karat IDE运行）

```java
import java.util.HashMap;
import java.util.Map;

public class Solution {
    public int[] twoSum(int[] nums, int target) {
        if (nums == null || nums.length < 2) {
            return new int[]{};
        }
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[]{map.get(complement), i};
            }
            map.put(nums[i], i);
        }
        return new int[]{};
    }
}
```

---

## Question 2：Contains Duplicate 数组重复元素（Karat 超高频｜简单但必考工程思维）

### 1\. English Problem

Given an integer array nums, return true if any value appears at least twice in the array, and return false if every element is distinct\.

### 2\. Full English Speaking Script（全程口述）

**Restate**: The problem is to check whether there are duplicate elements in the array\. If duplicates exist, return true; otherwise return false\.

**Edge Case**: If the array length is 0 or 1, there must be no duplicates, return false directly\.

**Solution Idea**: I can use a HashSet to record elements I have traversed\. As I iterate through the array, if the current element is already in the set, it means we find a duplicate, return true immediately\. If not, add the element to the set\. If the loop finishes without duplicates, return false\.

**Complexity**: Time complexity is O\(n\), space complexity is O\(n\)\.

**Test Case**: For nums = \[1,2,3,1\], the last element 1 is already in the set, so we return true\. For nums = \[1,2,3\], no duplicates, return false\.

### 3\. Java Full Code

```java
import java.util.HashSet;
import java.util.Set;

public class Solution {
    public boolean containsDuplicate(int[] nums) {
        if (nums == null || nums.length <= 1) {
            return false;
        }
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            if (set.contains(num)) {
                return true;
            }
            set.add(num);
        }
        return false;
    }
}
```

---

## Question 3：Best Time to Buy and Sell Stock 买卖股票（Karat 中等高频｜遍历思维必考）

### 1\. English Problem

You are given an array prices where prices\[i\] is the price of a given stock on the ith day\. You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock\. Return the maximum profit you can achieve from this transaction\. If you cannot achieve any profit, return 0\.

### 2\. Full English Speaking Script（全程口述）

**Restate**: The rule is buy first and then sell later\. We need to find the maximum profit\. If no profit is possible, return 0\.

**Edge Case**: If the array length is less than 2, we cannot complete buy and sell, so return 0\.

**Solution Idea**: I will track the minimum price during traversal\. For each day’s price, calculate the current profit by subtracting the minimum price\. Update the maximum profit every time\. This only needs one single traversal\.

**Complexity**: Time complexity is O\(n\), space complexity is O\(1\), no extra collection space\.

**Test Case**: For prices = \[7,1,5,3,6,4\]\. The minimum price starts at 7, then updates to 1\. The max profit is 6 minus 1, which is 5\. So the answer is 5\.

### 3\. Java Full Code

```java
public class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length < 2) {
            return 0;
        }
        int minPrice = prices[0];
        int maxProfit = 0;
        for (int i = 1; i < prices.length; i++) {
            minPrice = Math.min(minPrice, prices[i]);
            maxProfit = Math.max(maxProfit, prices[i] - minPrice);
        }
        return maxProfit;
    }
}
```

---

## Karat 三道题通用满分口诀（考前必看）

1\. Always confirm the problem before coding
2\. Speak edge cases actively
3\. Explain time \& space complexity clearly
4\. Walk through test cases after coding
5\. Keep talking, no silence

> （注：部分内容可能由 AI 生成）
