# Maximum Average Subarray I

**LeetCode Link:** https://leetcode.com/problems/maximum-average-subarray-i/

**Difficulty:** Easy

**Study Plan:** LeetCode 75 — Sliding Window

## Problem Statement

You are given an integer array `nums` consisting of `n` elements, and an integer `k`.

Find a contiguous subarray whose length is equal to `k` that has the maximum average value and return this value. Any answer with a calculation error less than `10^-5` will be accepted.

**Example 1:**
```
Input: nums = [1,12,-5,-6,50,3], k = 4
Output: 12.75000
Explanation: Maximum average is (12 - 5 - 6 + 50) / 4 = 51 / 4 = 12.75
```

**Example 2:**
```
Input: nums = [5], k = 1
Output: 5.00000
```

**Constraints:**
- `n == nums.length`
- `1 <= k <= n <= 10^5`
- `-10^4 <= nums[i] <= 10^4`

## Approach 1: Brute Force (for intuition)

Every window of length `k` starts at some index `i` in `[0, n - k]`. Compute each window's sum from scratch and keep the largest.

```cpp
double findMaxAverage(vector<int>& nums, int k) {
    int n = nums.size();
    double best = -1e9;

    for (int i = 0; i + k <= n; i++) {
        double sum = 0;
        for (int j = i; j < i + k; j++) sum += nums[j];
        best = max(best, sum / k);
    }
    return best;
}
```

**Time Complexity:** O(n·k) — up to `n - k + 1` windows, each summed in O(k).
**Space Complexity:** O(1).

With `n = 10^5` and `k` near `n` this is ~10^9 operations — too slow. The waste is obvious: consecutive windows overlap in `k - 1` elements, so almost every addition is repeated.

## Approach 2: Sliding Window (Optimal)

Since the window has a **fixed** size, moving it one step right only changes two elements: one enters on the right, one leaves on the left. Keep a running sum and patch it in O(1) per step instead of recomputing.

**Idea:**
- Build the sum of the first `k` elements — this is the initial window.
- Slide `i` from `k` to `n - 1`: add `nums[i]` (entering) and subtract `nums[i - k]` (leaving).
- Track the maximum **sum** seen, and divide by `k` only once at the end. Comparing sums instead of averages avoids repeated floating-point division and keeps the comparison exact in integers.

```cpp
class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        long long sum = 0;

        for (int i = 0; i < k; i++) sum += nums[i];

        long long best = sum;

        for (int i = k; i < (int)nums.size(); i++) {
            sum += nums[i] - nums[i - k];   // element enters, element leaves
            best = max(best, sum);
        }

        return (double)best / k;
    }
};
```

**Time Complexity:** O(n) — each element enters the window once and leaves once.
**Space Complexity:** O(1) — only the running sum and best sum are stored.

**Note on types:** `k <= 10^5` and `|nums[i]| <= 10^4`, so a window sum reaches `10^9` in magnitude — that still fits in a 32-bit `int`, but `long long` removes any doubt and costs nothing here. The final division must be floating point: cast before dividing, otherwise integer division truncates.

## Walkthrough with nums = [1,12,-5,-6,50,3], k = 4

Initial window `[1,12,-5,-6]` → sum = 2, best = 2.

| i | enters | leaves | sum | best |
|---|---|---|---|---|
| 4 | 50 | 1 | 2 + 50 - 1 = 51 | 51 |
| 5 | 3 | 12 | 51 + 3 - 12 = 42 | 51 |

**Result:** `51 / 4 = 12.75`.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n·k) | O(1) | Re-sums overlapping elements; TLE at the upper limits |
| Sliding Window | O(n) | O(1) | Optimal; accepted solution above |

Fixed-size sliding window is the base pattern of the LeetCode 75 sliding-window section: maintain one aggregate over the window and repair it in O(1) as the window advances. The same skeleton solves *Maximum Number of Vowels in a Substring of Given Length* (aggregate = vowel count) and generalizes to variable-size windows such as *Max Consecutive Ones III*, where the right edge always advances but the left edge only moves while a constraint is violated.