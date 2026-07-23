# Longest Substring Without Repeating Characters

**LeetCode Link:** [https://leetcode.com/problems/longest-substring-without-repeating-characters/](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

**Difficulty:** Medium

## Problem Statement

Given a string `s`, find the length of the **longest substring** without repeating characters.

**Example 1:**
```
Input: s = "abcabcbb"
Output: 3
Explanation: The answer is "abc", with the length of 3.
```

**Example 2:**
```
Input: s = "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
```

**Example 3:**
```
Input: s = "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3.
Notice that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

---

## Approach 1: Brute Force (for intuition)

Check every possible substring and verify whether it has all unique characters.

```cpp
int lengthOfLongestSubstring(string s) {
    int n = s.length();
    int maxLength = 0;

    for (int i = 0; i < n; i++) {
        unordered_set<char> seen;
        for (int j = i; j < n; j++) {
            if (seen.count(s[j])) break;
            seen.insert(s[j]);
            maxLength = max(maxLength, j - i + 1);
        }
    }
    return maxLength;
}
```

**Time Complexity:** O(n²) — for every starting index `i`, we scan forward until a duplicate is found.
**Space Complexity:** O(min(n, charset)).

This works but is wasteful: it re-scans overlapping ranges instead of reusing what it already knows about the window.

---

## Approach 2: Sliding Window with Hash Map

Sliding window is the standard technique for "longest/shortest contiguous subrange satisfying a condition" problems. Instead of restarting from every index, maintain a window `[left, right]` that only ever expands or contracts forward — never resetting — so every character is processed a constant number of times.

**Idea:**
- `right` scans forward, expanding the window one character at a time.
- `charMap` remembers the most recent index at which each character was seen.
- If the incoming character `s[right]` was already seen **inside the current window** (i.e., at or after `left`), jump `left` directly past that earlier occurrence — no need to shrink one step at a time.
- Track the best window size (`right - left + 1`) seen so far.

```cpp
int lengthOfLongestSubstring(string s) {
    int n = s.length();
    int maxLength = 0;
    unordered_map<char, int> charMap;
    int left = 0;

    for (int right = 0; right < n; right++) {
        if (charMap.count(s[right]) == 0 || charMap[s[right]] < left) {
            charMap[s[right]] = right;
            maxLength = max(maxLength, right - left + 1);
        } else {
            left = charMap[s[right]] + 1;
            charMap[s[right]] = right;
        }
    }

    return maxLength;
}
```

**Why the `charMap[s[right]] < left` check matters:** without it, a character seen long before the current window started could incorrectly trigger a jump. The check ensures only duplicates *inside the active window* affect `left`.

**Time Complexity:** O(n) — `right` visits each index once; `left` only ever moves forward.
**Space Complexity:** O(min(n, charset)) for the hash map.

---

## Approach 3: Sliding Window with Fixed-Size Array (Optimized)

Same algorithm, but since characters have a bounded range (ASCII), a fixed-size array replaces the hash map. This avoids hashing overhead, collision handling, and heap allocation — all constant-factor wins that matter in practice, even though the Big-O stays the same.

```cpp
int lengthOfLongestSubstring(string s) {
    int n = s.length();
    int maxLength = 0;
    int left = 0;
    vector<int> lastSeen(256, -1);  // extended ASCII; use 128 for plain ASCII, 26 for lowercase-only

    for (int right = 0; right < n; right++) {
        char c = s[right];
        if (lastSeen[c] >= left) {
            left = lastSeen[c] + 1;
        }
        lastSeen[c] = right;
        maxLength = max(maxLength, right - left + 1);
    }

    return maxLength;
}
```

**Time Complexity:** O(n).
**Space Complexity:** O(1) — the array size is fixed (256 entries), independent of input size.

**Walkthrough with `s = "abba"`:**

| right | char | condition | left | window | maxLength |
|-------|------|-----------|------|--------|-----------|
| 0 | a | new | 0 | "a" | 1 |
| 1 | b | new | 0 | "ab" | 2 |
| 2 | b | dup at 1 ≥ left(0) | 2 | "b" | 2 |
| 3 | a | last seen at 0 < left(2) → treated as new | 2 | "ba" | 2 |

Result: **2** (`"ab"` or `"ba"`).

---

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(min(n, charset)) | Simple but re-scans overlapping work |
| Sliding Window + Hash Map | O(n) | O(min(n, charset)) | Standard interview-ready solution |
| Sliding Window + Array | O(n) | O(1) | Best constant factor; ideal when charset is bounded |

The sliding-window pattern generalizes to many problems: longest substring with at most K distinct characters, minimum window substring, longest subarray with sum ≤ K, and more. The core idea is always the same — expand the window with a `right` pointer, and only shrink from `left` when a constraint is violated, never resetting from scratch.
