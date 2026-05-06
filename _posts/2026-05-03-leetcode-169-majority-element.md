---
title: "169 · Majority Element"
date: 2026-05-03 17:15:00 +0000
categories: [Algorithms, LeetCode]
tags: [array, hashmap, greedy, java, interview, leetcode, boyer-moore]
description: "From counting with a HashMap to Boyer–Moore voting: all main approaches to LeetCode 169, why they work, and when to use each in an interview."
---

## What Is This Problem Really About?

The formal task: given an array `nums`, find the **majority element** — the element that appears **more than n/2 times**. The problem **guarantees** that such an element always exists.

The deeper goal is not just to find that element but to recognize that:

- A straightforward **frequency count** works fine.
- The special **> n/2 guarantee** allows a much more space-efficient solution.
- You should be able to move from the naive O(n)–space approach to an O(1)–space solution when prompted in an interview.

This is where the **Boyer–Moore majority vote algorithm** comes in.

***

## Problem Classification

| Property | Detail |
|---|---|
| **Pattern** | Counting / Voting on arrays |
| **Core goal** | Find the majority element (> n/2 occurrences) |
| **Key guarantee** | Majority element always exists |
| **Baseline solution** | HashMap frequency count (O(n) time, O(n) space) |
| **Optimal solution** | Boyer–Moore (O(n) time, O(1) space) |

> The guarantee that a majority element **must exist** is what makes Boyer–Moore possible without a second verification pass.

***

## Approach 1 — HashMap Frequency Count (O(n) time, O(n) space)

This is the most natural way to think about the problem:

1. Traverse the array and maintain a **frequency map** from value → count.
2. Return the element whose count is greater than `n/2`.

This solution is easy to explain and is often where you should start in an interview.

```java
class Solution {
    public int majorityElement(int[] nums) {
        Map<Integer, Integer> freq = new HashMap<>();
        int n = nums.length;

        for (int num : nums) {
            int count = freq.getOrDefault(num, 0) + 1;
            freq.put(num, count);
            if (count > n / 2) {
                return num; // early exit once we find the majority
            }
        }

        // The problem guarantees a majority element exists,
        // so we never actually reach this line.
        return -1;
    }
}
```

**Pros:**

- Very easy to reason about.
- Great as a first answer under time pressure.

**Cons:**

- Uses **O(n) extra space** for the map.
- Interviewers will often ask: “Can you do it in O(1) space?”

***

## Approach 2 — Sorting (O(n log n) time, O(1) space)

If you sort the array, any element appearing more than `n/2` times must occupy the middle position index `n/2` after sorting. That gives a very short solution:

```java
import java.util.Arrays;

class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);                 // O(n log n)
        return nums[nums.length / 2];      // middle element is guaranteed majority
    }
}
```

**Pros:**

- Code is tiny.
- No extra data structure besides the array itself.

**Cons:**

- Time complexity is **O(n log n)** due to sorting, slower than the O(n) options.
- Still fine in practice, but not optimal.

This is a good “middle ground” answer if you can’t recall Boyer–Moore under pressure, but want to avoid a HashMap.

***

## Approach 3 — Boyer–Moore Majority Vote (O(n) time, O(1) space)

This is the optimal approach and the most interesting conceptually.

### Intuition

The key idea: since the majority element appears **more than half the time**, if you **pair up different elements and cancel them**, the majority element will still survive in the end.

Imagine “voting”:

- The majority element is a candidate with many supporters.
- Every time you see the same value as the candidate → it gains a vote.
- Every time you see a different value → that’s like one vote against it (they cancel out).

Even if many pairs cancel out, the true majority (more than n/2) cannot be completely eliminated. The element that survives this process is guaranteed to be the majority.

### Algorithm

Maintain:

- `candidate` — current majority candidate.
- `count` — the net “vote balance” for that candidate.

Rules as you scan from left to right:

1. If `count == 0` → set `candidate = num`, `count = 1`.
2. Else if `num == candidate` → `count++`.
3. Else → `count--`.

At the end, `candidate` is the majority element.

```java
class Solution {
    public int majorityElement(int[] nums) {
        int candidate = 0;
        int count = 0;

        for (int num : nums) {
            if (count == 0) {
                candidate = num;   // pick a new candidate
                count = 1;
            } else if (num == candidate) {
                count++;           // same as candidate → support
            } else {
                count--;           // different → cancel one vote
            }
        }

        // Because the problem guarantees that a majority element exists,
        // candidate must be that majority.
        return candidate;
    }
}
```

### Example Walkthrough

Take:

```text
nums = [2, 2, 1, 1, 1, 2, 2]
```

Step-by-step:

```text
candidate=?, count=0
num=2 → count==0 → candidate=2, count=1
num=2 → same as candidate → count=2
num=1 → different → count=1
num=1 → different → count=0
num=1 → count==0 → candidate=1, count=1
num=2 → different → count=0
num=2 → count==0 → candidate=2, count=1

End: candidate=2 → majority element
```

Even though the candidate changes during the process, the final candidate must be the true majority because only that value can survive all the cancellations.

***

## Interview Strategy

When asked LC 169 in an interview, a good progression looks like this:

1. **Start with HashMap**  
   Explain the O(n) / O(n) approach — it shows clear, correct thinking.
2. **Mention sorting**  
   Short and easy: sort and return `nums[n/2]`.
3. **Then aim for Boyer–Moore**  
   Use the majority guarantee and the “pairwise cancellation” intuition to derive the O(n) / O(1) solution.

If you can communicate not only the code but also the *why* behind the Boyer–Moore voting, you’ll stand out.

***

## Summary of Approaches

| Approach | Time | Space | Idea |
|---|---|---|---|
| HashMap frequency | O(n) | O(n) | Count occurrences of each element |
| Sorting | O(n log n) | O(1) extra | Majority sits at index `n/2` after sort |
| Boyer–Moore | O(n) | O(1) | Cancel out non-majority elements with a vote counter |

