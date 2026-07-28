# Longest Palindromic Substring

**LeetCode Link:** https://leetcode.com/problems/longest-palindromic-substring/

**Difficulty:** Medium

## Problem Statement

Given a string `s`, return the longest palindromic substring in `s`.

**Example 1:**
```
Input: s = "babad"
Output: "bab"
Explanation: "aba" is also a valid answer.
```

**Example 2:**
```
Input: s = "cbbd"
Output: "bb"
```

## Approach 1: Brute Force (for intuition)

Check every possible substring and verify whether it reads the same forwards and backwards, keeping track of the longest one found.

```cpp
string longestPalindrome(string s) {
    int n = s.size();
    string result = "";

    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {
            string sub = s.substr(i, j - i + 1);
            bool isPal = true;
            for (int l = 0, r = sub.size() - 1; l < r; l++, r--) {
                if (sub[l] != sub[r]) { isPal = false; break; }
            }
            if (isPal && sub.size() > result.size()) result = sub;
        }
    }
    return result;
}
```

**Time Complexity:** O(n³) — O(n²) substrings, each checked in O(n).
**Space Complexity:** O(n) for the substring being tested.

This works but redoes a lot of comparison work that a smarter approach can reuse.

## Approach 2: Expand Around Center (Optimal)

A palindrome mirrors around its middle, so instead of checking every substring, treat every index (and every gap between two indices) as a potential center and grow outward while both sides match. There are `2n - 1` possible centers: `n` single-character centers (odd-length palindromes) and `n - 1` between-character centers (even-length palindromes).

**Idea:**
- For each index `i`, expand around `(i, i)` for odd-length palindromes and around `(i, i + 1)` for even-length palindromes.
- `expand` grows `left`/`right` outward while the characters match, then returns the last valid bounds.
- Track the widest `[start, end]` window seen across all centers.

```cpp
class Solution {
public:

    string longestPalindrome(string s) {
        if (s.empty()) return "";

        int start = 0, end = 0;

        for (int i = 0; i < s.size(); i++) {
            auto [l1, r1] = expand(s, i, i);       // odd length
            if (r1 - l1 > end - start) {
                start = l1;
                end = r1;
            }

            auto [l2, r2] = expand(s, i, i + 1);   // even length
            if (r2 - l2 > end - start) {
                start = l2;
                end = r2;
            }
        }

        return s.substr(start, end - start + 1);
    }

private:
    pair<int, int> expand(const string& s, int left, int right) {
        while (left >= 0 && right < (int)s.size() && s[left] == s[right]) {
            left--;
            right++;
        }
        return {left + 1, right - 1};
    }
};
```

**Time Complexity:** O(n²) — `2n - 1` centers, each expansion can take up to O(n).
**Space Complexity:** O(1) — only a few pointers are tracked, no extra data structures.

**Result:** Accepted, 144/144 testcases · Runtime 6ms (beats 88.53%) · Memory 9.36MB (beats 77.97%).

## Walkthrough with s = "babad"

| i | odd center expand | even center expand | best window so far |
|---|---|---|---|
| 0 | "b" | "" | "b" |
| 1 | "bab" | "" | "bab" |
| 2 | "aba" | "" | "bab" (tie, first found kept) |
| 3 | "a" | "" | "bab" |
| 4 | "d" | "" | "bab" |

**Result:** `"bab"` (equally valid: `"aba"`).

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n³) | O(n) | Simple but re-checks overlapping substrings |
| Expand Around Center | O(n²) | O(1) | Optimal constant space; accepted solution above |

Expanding around centers is a recurring palindrome-problem pattern: it also underlies palindrome partitioning and counting all palindromic substrings, where the same `expand` helper is reused as a building block.
