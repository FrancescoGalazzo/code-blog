---
title: "Prefix Sums vs Sliding Window: What If LC 209 Had Negative Numbers?"
date: 2026-05-04 11:30:00 +0000
categories: [Algorithms, LeetCode]
tags: [prefix-sum, sliding-window, array, interview, leetcode, deque]
description: "Why the classic sliding window for LeetCode 209 breaks with negative numbers, and the correct O(n) approach using a monotonic deque."
---

## 1. Problem Setup: LC 209, But Harder

The original **LeetCode 209** states:

> Given a **positive** integer array `nums` and a positive integer `target`, return the minimal length of a contiguous subarray whose sum is **greater than or equal to** `target`. If no such subarray exists, return `0`.

When all `nums[i] > 0`, the optimal solution is a **variable-size sliding window** with **O(n)** time complexity.

### The Harder Variant

Now imagine a small but critical change:

> `nums` can contain **negative numbers** as well.

Everything else stays the same: contiguous subarray, sum ≥ `target`, minimize length.

This tiny modification **completely breaks** the sliding window approach. This post explains:

1. Why sliding window **depends on positivity** and fails with negatives
2. How **prefix sums** transform the problem into a searchable inequality
3. The correct **O(n)** solution using a monotonic deque

---

## 2. Why Sliding Window Needs Positive Numbers

### The Monotonicity Property

For the classic problem (all positives), sliding window works because of **monotonicity**:

| Operation | Effect on Window Sum |
|-----------|---------------------|
| Expand right: `sum += nums[right]` | Sum **never decreases** |
| Shrink left: `sum -= nums[left]` | Sum **never increases** |

This enables a clean greedy strategy:

```java
int left = 0, sum = 0, answer = INF;
for (int right = 0; right < n; right++) {
    sum += nums[right];                    // Expand
    while (sum >= target) {                // Shrink while valid
        answer = Math.min(answer, right - left + 1);
        sum -= nums[left++];
    }
}
```

**Why O(n)?** Each index enters and leaves the window at most once.

### Why Negatives Break This

With negative numbers:

| Operation | Possible Effect |
|-----------|----------------|
| Add `nums[right] < 0` | Sum **decreases** ❌ |
| Remove `nums[left] < 0` | Sum **increases** ❌ |

The rule *"while sum ≥ target, shrink from left"* may:
- Discard a valid starting point prematurely
- Miss shorter solutions that appear later
- Require re-expanding the window, breaking the O(n) guarantee

**Conclusion**: When negatives are allowed, we need a different paradigm: **prefix sums**.

---

## 3. Prefix Sums: The Core Transformation

### Definition

Define the prefix sum array `prefix` of length `n + 1`:

- `prefix[0] = 0`
- For `k ≥ 1`:
  ```
  prefix[k] = nums[0] + nums[1] + ... + nums[k-1]
  ```

So `prefix[k]` represents the sum of all elements **before index `k`**.

### Example

```
nums   = [2, -1,  2, -2,  3]
index     0   1   2   3   4

prefix[0] = 0
prefix[1] = 0 + 2         = 2
prefix[2] = 2 + (-1)      = 1
prefix[3] = 1 + 2         = 3
prefix[4] = 3 + (-2)      = 1
prefix[5] = 1 + 3         = 4

prefix = [0, 2, 1, 3, 1, 4]
index    0  1  2  3  4  5
```

### Key Identity: Subarray Sum via Prefix Differences

For any subarray `nums[i..j]` (inclusive):

```
sum(nums[i..j]) = prefix[j+1] - prefix[i]
```

**Proof**:
- `prefix[j+1]` = sum of `nums[0..j]`
- `prefix[i]` = sum of `nums[0..i-1]`
- Subtracting cancels the common prefix, leaving `nums[i..j]` ✓

> 🔑 **Subarray sum = difference of two prefix sums**

This identity lets us rewrite any subarray-sum condition as an inequality on prefix values.

---

## 4. Reformulating the Problem

We want the **shortest** subarray `nums[i..j]` such that:

```
sum(nums[i..j]) >= target
```

Using prefix sums:

```
prefix[j+1] - prefix[i] >= target
```

Let:
- `end = j + 1` (prefix index *after* the subarray ends)
- `start = i` (prefix index *before* the subarray starts)

The inequality becomes:

```
prefix[end] - prefix[start] >= target
⇔
prefix[start] <= prefix[end] - target
```

### Restated Goal

For each `end ∈ [1, n]`, find a `start < end` such that:

1. `prefix[start] ≤ prefix[end] - target`  (valid subarray)
2. `end - start` is **minimized** (shortest length)

Equivalently: **maximize `start`** among all valid candidates.

> 🎯 The problem reduces to: *As we scan `end`, quickly find the largest `start < end` with `prefix[start] ≤ need`, where `need = prefix[end] - target`.*

---

## 5. The Optimal Solution: Monotonic Deque (O(n))

### Key Insight

Maintain a deque of candidate `start` indices such that their prefix values are **strictly increasing**. This guarantees:

1. **Front of deque**: smallest prefix value → best chance to satisfy `prefix[end] - prefix[start] ≥ target`
2. **Back of deque**: newest indices with small values dominate older ones

### Algorithm Steps

For each `end` from `0` to `n`:

1. **Check validity**: While `prefix[end] - prefix[deque.front] ≥ target`, we found a valid subarray. Pop front and update answer (greedy: popping front gives shortest length for the current `end`).
2. **Maintain monotonicity**: While `prefix[deque.back] ≥ prefix[end]`, pop back. A newer index with a smaller or equal prefix value is always preferable — it can only produce a shorter subarray.
3. **Add current**: Push `end` to the back of the deque.

### Complete Implementation

```java
import java.util.*;

public class MinSubarrayWithNegatives {

    /**
     * Returns minimal length of contiguous subarray with sum >= target.
     * Works with negative numbers. O(n) time, O(n) space.
     */
    public int minSubArrayLen(int target, int[] nums) {
        int n = nums.length;
        long[] prefix = new long[n + 1];

        // Build prefix sums (use long to prevent overflow)
        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        Deque<Integer> deque = new ArrayDeque<>();
        int answer = Integer.MAX_VALUE;

        for (int end = 0; end <= n; end++) {
            // 1) Greedily shrink from front while valid
            while (!deque.isEmpty() && prefix[end] - prefix[deque.peekFirst()] >= target) {
                answer = Math.min(answer, end - deque.pollFirst());
            }

            // 2) Maintain strictly increasing prefix values in deque
            while (!deque.isEmpty() && prefix[deque.peekLast()] >= prefix[end]) {
                deque.pollLast();
            }

            // 3) Add current index as future candidate start
            deque.offerLast(end);
        }

        return answer == Integer.MAX_VALUE ? 0 : answer;
    }
}
```

### Why This Handles Negatives Correctly

| Step | Purpose | Handles Negatives Because... |
|------|---------|-----------------------------|
| Prefix sums | Convert subarray sum to difference | Works for any numbers |
| Deque front removal | Find shortest valid subarray ending at `end` | Checks *all* valid starts greedily |
| Deque back removal | Keep only "useful" candidates | Newer index with ≤ prefix value dominates older ones, even with negatives |
| Monotonic invariant | Prefix values strictly increase in deque | Ensures no better candidate is ever discarded |

### Walkthrough: `nums = [2, -1, 2, -2, 3]`, `target = 3`

| end | prefix[end] | deque (indices) | Action | answer |
|-----|-------------|-----------------|--------|--------|
| 0 | 0 | [] | Push 0 | ∞ |
| 1 | 2 | [0] | Push 1 | ∞ |
| 2 | 1 | [0] | Pop 1 (prefix≥current), push 2 | ∞ |
| 3 | 3 | [0,2] | 3-0=3≥3 → pop 0, length=3; push 3 | **3** |
| 4 | 1 | [2,3] | Pop 3 (prefix≥current), push 4 | 3 |
| 5 | 4 | [2,4] | 4-1=3≥3 → pop 2, length=3; 4-1=3≥3 → pop 4, length=1 | **1** ✓ |

**Result**: Minimal length = `1` (subarray `[3]`).

---

## 6. Complete Test Suite

```java
public class SolutionTests {
    public static void main(String[] args) {
        MinSubarrayWithNegatives solver = new MinSubarrayWithNegatives();

        assert solver.minSubArrayLen(3,   new int[]{2,-1,2,-2,3})  == 1 : "Test 1 failed";
        assert solver.minSubArrayLen(3,   new int[]{1,-1,5,-2,3})  == 1 : "Test 2 failed";
        assert solver.minSubArrayLen(1,   new int[]{-1,-2,-3})     == 0 : "Test 3 failed";
        assert solver.minSubArrayLen(7,   new int[]{2,3,1,2,4,3})  == 2 : "Test 4 failed";
        assert solver.minSubArrayLen(5,   new int[]{5})             == 1 : "Test 5 failed";
        assert solver.minSubArrayLen(10,  new int[]{})              == 0 : "Test 6 failed";
        assert solver.minSubArrayLen(4,   new int[]{1,-1,1,-1,5})  == 1 : "Test 7 failed";
        assert solver.minSubArrayLen(100, new int[]{1,2,3})         == 0 : "Test 8 failed";
        assert solver.minSubArrayLen(2,   new int[]{-5,10,-3,1})   == 1 : "Test 9 failed";
        assert solver.minSubArrayLen(4,   new int[]{1,1,1,1})       == 4 : "Test 10 failed";
        assert solver.minSubArrayLen(5,   new int[]{3})             == 0 : "Test 11 failed";
        assert solver.minSubArrayLen(5,   new int[]{-1,6,-1})      == 1 : "Test 12 failed";
        assert solver.minSubArrayLen(1,   new int[]{3,-1,2})        == 1 : "Test 13 failed";
        assert solver.minSubArrayLen(15,  new int[]{-100,10,5})    == 2 : "Test 14 failed";

        System.out.println("✅ All tests passed!");
    }
}
```

---

## 7. Key Takeaways

1. **Sliding window requires monotonicity** → fails with negative numbers.
2. **Prefix sums transform subarray problems** into prefix-difference inequalities.
3. **Monotonic deque** solves the generalized problem in **O(n)** — both optimal and production-ready.
4. **Always use `long` for prefix sums** to avoid integer overflow.
5. **Test edge cases**: empty arrays, all negatives, duplicate prefix values, single elements.

---

## 8. When to Use Which Pattern

| Problem Variant | Recommended Approach |
|----------------|---------------------|
| All positives, sum ≥ target | Sliding window O(n) |
| Negatives allowed, sum ≥ target | **Monotonic deque O(n)** |
| Sum = target (exact) | Hash map of prefix sums O(n) |
| Max sum subarray | Kadane's algorithm O(n) |
| Count subarrays with sum in range | Prefix sums + BST/Fenwick O(n log n) |

> 💡 **Rule of thumb**: If the problem involves *contiguous subarray sums* with *no positivity guarantee*, reach for **prefix sums + monotonic deque**.

---

## Appendix: Full Production-Ready Code

```java
import java.util.*;

/**
 * Solves: Minimal length of contiguous subarray with sum >= target.
 * Handles negative numbers correctly. O(n) time, O(n) space.
 */
public class MinSubarrayWithNegatives {

    public int minSubArrayLen(int target, int[] nums) {
        if (nums == null || nums.length == 0) return 0;

        int n = nums.length;
        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        Deque<Integer> deque = new ArrayDeque<>();
        int answer = Integer.MAX_VALUE;

        for (int end = 0; end <= n; end++) {
            // Shrink from front while we have valid subarrays
            while (!deque.isEmpty() && prefix[end] - prefix[deque.peekFirst()] >= target) {
                answer = Math.min(answer, end - deque.pollFirst());
            }

            // Maintain strictly increasing prefix values
            while (!deque.isEmpty() && prefix[deque.peekLast()] >= prefix[end]) {
                deque.pollLast();
            }

            deque.offerLast(end);
        }

        return answer == Integer.MAX_VALUE ? 0 : answer;
    }
}
```

