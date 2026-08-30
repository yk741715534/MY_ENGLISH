# Karat 全真60分钟英文面试完整脚本（Java版·可直接背诵）

适用场景：全英文技术面试、Java 技术栈、Karat 第三方标准化面试
核心优势：完全贴合 Karat 评分逻辑（**优先看英文表达、思考过程、边界Case、复杂度、自测**）

## Part 1\. Opening 开场（0–5min）

**Interviewer**: Could you please introduce yourself briefly?

**You（标准应答）**: Sure\. I’m a backend developer focused on Java development\. I have solid experience with Java core concepts, data structures and algorithms\. I’m also familiar with object\-oriented programming and common Java collections\. I’m glad to participate in today’s technical interview\.

**Interviewer**: Do you have any experience working with algorithms in your daily development?

**You**: Yes\. I usually use common data structures such as array, HashMap and linked list to optimize business logic\. I also practice algorithm problems regularly to improve my problem\-solving ability\.

## Part 2\. Java Core 英文问答（高频必考，5–20min）

**Q1: What’s the difference between ArrayList and LinkedList?**

**You**: ArrayList is based on a dynamic array\. It supports fast random access by index, but insertion and deletion in the middle are slow because elements need to be shifted\.
LinkedList is a doubly linked list\. It has slow random access, but it’s more efficient for frequent insertion and deletion at head and tail\.
In most business scenarios, ArrayList is preferred\.

**Q2: How does Java HashMap work?**

**You**: HashMap stores key\-value pairs\. It uses the key’s hash code to calculate the bucket index\.
When hash collision occurs, elements are stored in a linked list\. Since Java 8, if the linked list length exceeds the threshold, it will be converted to a red\-black tree to improve query performance\.
Besides, custom keys must override equals and hashCode methods correctly\.

**Q3: String vs StringBuilder vs StringBuffer?**

**You**: String is immutable\. Every modification creates a new string object, which causes poor performance for frequent concatenation\.
StringBuilder is mutable and not thread\-safe\. It’s used for single\-thread string splicing with better performance\.
StringBuffer is mutable and thread\-safe with synchronized methods, so it’s slower\.

**Q4: What’s the difference between Heap and Stack in JVM?**

**You**: Stack is thread private\. It stores local variables and method frames, and memory is released after method execution\.
Heap is thread shared\. It stores all objects and arrays\. Garbage collection mainly works on the heap to recycle unused memory\.

**Q5: == vs equals\(\)**

**You**: For primitive types, == compares values\. For reference types, == compares memory addresses\.
The equals method originally compares references, but common classes like String override it to compare actual content\.

**Q6: What are the four OOP principles?**

**You**: There are four core principles\.
First, Encapsulation, which hides internal data and exposes public methods\.
Second, Inheritance, which allows child classes to reuse parent class code\.
Third, Polymorphism, which enables different behaviors for different subclasses\.
Fourth, Abstraction, which hides complex implementation details\.

## Part 3\. Algorithm Coding 全真大题（核心打分项 20–50min）

**Interviewer Question**: Given an integer array nums and a target value, return indices of two numbers that add up to target\. Each input has exactly one solution, and you cannot reuse the same element\.

### Step1: Restate \& Confirm（必须说，Karat 加分项）

**You**: Let me confirm the problem\. I need to find two distinct elements in the array whose sum equals the target, and return their indices\. And there is exactly one valid answer\. Is that correct?

### Step2: Discuss Edge Cases（必说）

**You**: First I want to consider edge cases\. If the input array is null or its length is less than 2, we cannot get two numbers, so we return an empty array\.

### Step3: Analyze Solution \& Complexity（高分关键）

**You**: The brute force solution uses two nested loops, but its time complexity is O\(n²\), which is not efficient\.
I can optimize it with HashMap\. The idea is to store each number and its index while traversing\.
For each current number, I calculate the complement value which equals target minus current number\.
If the complement exists in the map, we find the answer\. If not, we put current value and index into the map\.
The time complexity is O\(n\), and space complexity is O\(n\) for the HashMap\.

### Step4: Write Java Code（可直接默写）

```java
import java.util.HashMap;
import java.util.Map;

public class Solution {
    public int[] twoSum(int[] nums, int target) {
        // handle edge case
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

### Step5: Walk Through Test Case（必须口述，Karat 核心评分点）

**You**: Let me test with sample input\. If nums is \[2,7,11,15\] and target is 9\.
At index 0, number is 2, complement is 7\. The map does not have 7, so we put \(2,0\)\.
At index 1, number is 7, complement is 2\. The map contains 2 with index 0\.
So we return \[0,1\], which matches the correct answer\.

### Step6: Final Self\-check（收尾加分）

**You**: This solution handles empty input, runs in linear time, and meets the problem requirements\.

## Part 4\. Candidate Q\&A 结尾提问（5min）

**Interviewer**: Do you have any questions for me?

**You（优选2选1，专业得体）**

Option1: Could you please share what technical challenges the team is currently focusing on?

Option2: What kind of engineer performance does your team value most?

## ✅ Karat 面试独家得分口诀（考前默念）

1\. Always talk, never silent（哪怕慢，不要沉默）
2\. Confirm problem first, then code（先确认题目）
3\. State edge cases actively（主动说边界）
4\. Explain time \& space complexity（必讲复杂度）
5\. Walk through test case after coding（写完必自测）

> （注：部分内容可能由 AI 生成）
