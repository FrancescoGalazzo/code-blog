---
title: "Kadane's vs Sliding Window — When the Pattern Changes"
date: 2026-04-30 12:00:00 +0000
categories: [Algorithms, LeetCode]
tags: [greedy, sliding-window, array, java, interview, leetcode, kadane, hashset]
description: How a tiny change in constraints turns a Kadane's problem into a Sliding Window problem, and the rule of thumb to know which pattern to use.
---

If you haven't seen Kadane's Algorithm yet, you can start with my post on
[LeetCode 53 · Maximum Subarray](/code-blog/posts/leetcode-53-maximum-subarray/).

## From Simple to Constrained: How the Pattern Changes

Start with this classic problem:

> **Constraint 1:** “Find the maximum sum of any contiguous subarray.”

This is LeetCode 53. There is **no restriction** on the elements inside the subarray — you only care about the sum. Kadane’s Algorithm fits perfectly here.

Now change just one line in the problem:

> **Constraint 2:** “Find the maximum sum of any contiguous subarray whose elements are all **unique**.”

The optimization goal is the same (maximum sum), but the **constraint on the subarray** is now different. That small change is enough to break Kadane’s and force a completely different pattern: **Sliding Window with a set/map**.

Understanding this jump — from Constraint 1 → Constraint 2, and from Kadane’s → Sliding Window — is the real goal of this post.

***

## A Quick Recap of Kadane's Algorithm (Constraint 1)

For the unconstrained maximum subarray problem, Kadane’s works by carrying a single running variable — `currentSum` — forward through the array. At every index you make one binary decision:

- `currentSum > 0` → **extend**: the past is helping, keep it
- `currentSum ≤ 0` → **start fresh**: the past is hurting, cut it

The reset condition is purely **numerical**: one number (`currentSum`) summarizes everything you need to know about the past. No other information is required.

```java
for (int num : nums) {
    if (currentSum > 0) currentSum += num;
    else                currentSum = num;
    maxSum = Math.max(maxSum, currentSum);
}
```

This works because, under Constraint 1, the **only** thing that matters about a past subarray is whether its sum is positive or negative. The actual elements don’t matter.

***

## Why Adding Uniqueness Breaks Kadane's (Constraint 2)

Now add the new requirement:

> “All elements in the subarray must be **distinct**.”

The core question you need to answer at each step is no longer:

> *“Is my running sum negative?”*

It becomes:

> *“Does this element already exist somewhere in my current window?”*

That’s a **membership question**, not a numeric one.

Kadane’s carries exactly one integer — `currentSum`. That integer tells you nothing about **which elements** are currently in your subarray. You cannot answer a membership question with a single number.

```text
Kadane's knows:       currentSum = 6       (a number)
Kadane's doesn't know: which elements make up that sum
```

As soon as the validity of extending the window depends on the **contents** of the window (distinctness, frequencies, counts), you need a data structure to track those contents. That is exactly what the Sliding Window pattern provides.

***

## The New Pattern: Sliding Window + HashSet (Unique Elements Variant)

With the uniqueness constraint, the problem becomes:

> “Find the maximum sum of a contiguous subarray whose elements are all distinct.”

The pattern changes to:

- Maintain two pointers, `i` (left) and `j` (right), defining a window `[i, j]`.
- Maintain a `HashSet` of the elements currently inside the window.
- At each step, ask: *"Is `nums[j]` already in the set?"*

Rules:

- **If not present** → safe to expand:
  - Add `nums[j]` to the set
  - Add it to `currentSum`
  - Update `maxSum`
  - Move `j` right
- **If present** → window invalid:
  - Remove `nums[i]` from the set
  - Subtract it from `currentSum`
  - Move `i` right (shrink from the left) until the duplicate is gone

```java
public int maxSumUniqueSubarray(int[] nums) {
    int i = 0, j = 0;
    int currentSum = 0, maxSum = 0;
    Set<Integer> seen = new HashSet<>();

    while (j < nums.length) {
        if (!seen.contains(nums[j])) {
            seen.add(nums[j]);
            currentSum += nums[j];
            maxSum = Math.max(maxSum, currentSum);
            j++;
        } else {
            seen.remove(nums[i]);
            currentSum -= nums[i];
            i++;
        }
    }

    return maxSum;
}
```

**Complexity:**  
O(n) time — each element enters and leaves the window at most once.  
O(n) space — the set holds at most `n` elements.

The key difference from Kadane’s: the **reset** is no longer about a negative sum; it’s about an invalid window (a duplicate).

***

## Step-by-step Trace (Sliding Window in Action)

```text
nums =[1][2][3][4][5]

→ Expand j=0: window=         seen={1}         sum=1[1]
→ Expand j=1: window=       seen={1,2}       sum=3[2][1]
→ Expand j=2: window=     seen={1,2,3}     sum=6[3][2][1]
→ 3 duplicate at j=3!
    Shrink i=0: window=     seen={2,3}       sum=5[2][3]
    3 still duplicate? yes (at j=3)
    Shrink i=1: window=       seen={3}         sum=3[3]
    3 still duplicate? yes (at j=3)
    Shrink i=2: window=[]        seen={}          sum=0
→ Expand j=3: window=         seen={3}         sum=3[3]
→ Expand j=4: window=       seen={3,4}       sum=7[4][3]
→ Expand j=5: window=     seen={3,4,5}     sum=12[5][4][3]
→ Expand j=6: window=   seen={3,4,5,2}   sum=14[4][5][2][3]
→ Expand j=7: window= seen={3,4,5,2,1} sum=15  ✅[5][1][2][4][3]

Answer: 15  →  subarray[1][2][4][5][3]
```

Kadane’s cannot express this behavior, because it never tracks *which* elements are inside the subarray, only their total sum.

***

## The Root Difference: Constraint → Pattern

Both patterns scan the array in O(n). What changes is the **constraint** and therefore the **reset condition**:

| | Kadane's Algorithm | Sliding Window + HashSet |
|---|---|---|
| **Constraint** | Any subarray, maximize sum | Subarray sum with all distinct elements |
| **Reset question** | *Is the past helping?* | *Is the window still valid?* |
| **Reset trigger** | `currentSum ≤ 0` (value-based) | Duplicate discovered (membership-based) |
| **Past summarized by** | One number | Full window contents (set/map) |
| **Extra space** | O(1) | O(n) |
| **Pointers** | One (current index) | Two (`i` left, `j` right) |

So the story is:

- Start: **no content constraint** → Kadane’s is enough.
- Add: **uniqueness constraint** → must track contents → Sliding Window.

The pattern changes *because* the constraint changed.

***

## The Pattern-Recognition Rule

This is the interview rule to internalize:

> **If the reset condition depends on the *value* of a running aggregate** (sum, product, etc.) → use **Kadane's / Greedy**.
>
> **If the reset condition depends on the *contents* of the window** (distinct elements, counts, frequencies) → use **Sliding Window + HashSet / HashMap**.

### Keyword signals for Sliding Window

When you see these words in a subarray/substring problem, Sliding Window is almost always the right tool:

- “distinct” or “unique” elements
- “no repeating characters”
- “at most k duplicates”
- “exactly k different integers”
- “frequency of any element ≤ k”

Any condition that forces you to track **what is inside the window**, not just a numerical property of it, pushes you away from Kadane’s and into Sliding Window territory.

***

## The Full Pattern Family: How We Got Here

Reading these problems in order shows how constraints evolve and patterns change:

| Problem | Constraint | Pattern |
|---|---|---|
| LC 53 – Maximum Subarray | Any subarray, maximize sum | Kadane's Algorithm |
| LC 121 – Stock Buy & Sell | One transaction, maximize profit | Greedy running min |
| Unique elements variant | All distinct, maximize sum | Sliding Window + HashSet |
| LC 2461 – Max Sum Distinct Subarrays (Length K) | Fixed length `k`, all distinct | Fixed Sliding Window + HashMap |
| LC 2841 – Max Sum Almost Unique | Fixed length `k`, at least `m` distinct | Fixed Sliding Window + HashMap |
| LC 3 – Longest Substring Without Repeating Chars | No repeating characters | Sliding Window + HashSet |
| LC 992 – Subarrays with K Different Integers | Exactly `k` distinct | Sliding Window (advanced, counts) |

Each step adds or changes a **constraint** (distinctness, fixed length, “at least m”), and that is what drives the shift from **pure greedy/Kadane’s** to **Sliding Window with sets/maps**.