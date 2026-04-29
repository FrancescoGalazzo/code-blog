---
title: "Fibonacci Sequence — Java"
date: 2026-04-29 08:00:00 -0500
categories: [Java, Algorithms]
tags: [java, recursion, dynamic-programming, easy]
---

## The Problem

Return the `n`-th number in the Fibonacci sequence, where each number
is the sum of the two preceding ones.

**Sequence:** `0, 1, 1, 2, 3, 5, 8, 13, 21, ...`

**Example:**
- Input: `n = 6`
- Output: `8`

---

## Approach 1 — Recursion (Simple but Slow)

The most intuitive solution: `fib(n) = fib(n-1) + fib(n-2)`.

```java
public class Fibonacci {
    public int fib(int n) {
        if (n <= 1) return n;
        return fib(n - 1) + fib(n - 2);
    }
}
```

⚠️ **Problem:** Time complexity is **O(2ⁿ)** — it recalculates the
same values over and over. Very slow for large `n`.

---

## Approach 2 — Dynamic Programming (Fast)

Store already-computed values so we never calculate the same thing twice.

```java
public class Fibonacci {
    public int fib(int n) {
        if (n <= 1) return n;

        int[] dp = new int[n + 1];
        dp = 0;
        dp = 1;[1]

        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }
}
```

✅ Time complexity: **O(n)** | Space complexity: **O(n)**

---

## Approach 3 — Optimized DP (Best)

We only need the last two values — no need for the full array.

```java
public class Fibonacci {
    public int fib(int n) {
        if (n <= 1) return n;

        int prev2 = 0, prev1 = 1;

        for (int i = 2; i <= n; i++) {
            int curr = prev1 + prev2;
            prev2 = prev1;
            prev1 = curr;
        }

        return prev1;
    }
}
```

✅ Time complexity: **O(n)** | Space complexity: **O(1)**

---

## What I Learned

- Recursion is elegant but often hides terrible performance
- Dynamic programming trades memory for speed
- Always ask: *"Am I recalculating the same thing twice?"* — if yes,
  think DP
- The optimized version shows that sometimes you don't need to store
  the full history, just the last few steps