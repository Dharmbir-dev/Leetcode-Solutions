# Maximum Number of Vowels in a Substring of Given Length

**LeetCode Link:** https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/

**Difficulty:** Medium

**Study Plan:** LeetCode 75 — Sliding Window

## Problem Statement

Given a string `s` and an integer `k`, return the maximum number of vowel letters in any substring of `s` with length `k`.

Vowel letters are `'a'`, `'e'`, `'i'`, `'o'`, and `'u'`.

**Example 1:**
```
Input: s = "abciiidef", k = 3
Output: 3
Explanation: The substring "iii" contains 3 vowel letters.
```

**Example 2:**
```
Input: s = "aeiou", k = 2
Output: 2
Explanation: Any substring of length 2 contains 2 vowels.
```

**Example 3:**
```
Input: s = "leetcode", k = 3
Output: 2
Explanation: "lee", "eet" and "ode" contain 2 vowels.
```

**Constraints:**
- `1 <= s.length <= 10^5`
- `s` consists of lowercase English letters.
- `1 <= k <= s.length`

## Approach 1: Brute Force — recompute each window (TLE)

For each window start, slice out the substring and count its vowels from scratch.

```cpp
class Solution {
public:
    int maxVowels(string s, int k) {
        int sizeOfString = s.size();
        int best = countVowels(s.substr(0, k));

        for (int i = k; i < sizeOfString; i++) {
            int currentVowels = countVowels(s.substr(i - k + 1, k));
            if (currentVowels > best) {
                best = currentVowels;
            }
        }
        return best;
    }

private:
    int countVowels(string s) {
        int totalCount = 0;
        for (char c : s) {
            if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u') {
                ++totalCount;
            }
        }
        return totalCount;
    }
};
```

**Time Complexity:** O(n·k) — one full rescan of length `k` per window, for up to `n` windows.
**Space Complexity:** O(k) — each `substr` call allocates a new string.

**Note (why it TLEs):** `n` up to `10^5` and `k` up to `n` gives up to `~10^10` character comparisons in the worst case (e.g. `k` around `n/2`). Every window overlaps the previous one in `k - 1` characters, and this approach throws that overlap away and recounts it every time — the same waste pattern as the brute-force fixed window in *Maximum Average Subarray I*.

## Approach 2: Sliding Window (Optimal)

Window has fixed size `k`, so shifting it right by one only changes two characters: one enters on the right, one leaves on the left. Keep a running vowel `count` and patch it in O(1) per step.

**Idea:**
- At index `i`, `s[i]` is entering the window: if it's a vowel, `count++`.
- Once the window has grown past size `k` (`i >= k`), the character `s[i - k]` is leaving the window: if it's a vowel, `count--`.
- Once the window first reaches full size `k` (`i >= k - 1`), compare `count` against `best` — and keep comparing on every iteration after that, since the window stays full size for the rest of the scan.

```cpp
class Solution {
public:
    int maxVowels(string s, int k) {
        int sizeOfString = s.size();
        int count = 0, best = 0;

        for (int i = 0; i < sizeOfString; i++) {
            if (isVowel(s[i])) count++;
            if (i >= k) {
                if (isVowel(s[i-k])) count--;
            }
            if (i >= k-1) {
                best = max(best, count);
            }
        }
        return best;
    }

private:
    bool isVowel(char c) {
        return c=='a'||c=='e'||c=='i'||c=='o'||c=='u';
    }
};
```

**Time Complexity:** O(n) — each character is added to the window once and removed once.
**Space Complexity:** O(1) — only `count` and `best` are stored, no substring copies.

## Walkthrough with s = "leetcode", k = 3

| i | char | vowel? | count (after add) | remove s[i-k]? | count (after remove) | window full? | best |
|---|---|---|---|---|---|---|---|
| 0 | l | no | 0 | — | 0 | no | 0 |
| 1 | e | yes | 1 | — | 1 | no | 0 |
| 2 | e | yes | 2 | — | 2 | yes | 2 |
| 3 | t | no | 2 | s[0]='l' no | 2 | yes | 2 |
| 4 | c | no | 2 | s[1]='e' yes → -- | 1 | yes | 2 |
| 5 | o | yes | 2 | s[2]='e' yes → -- | 1 | yes | 2 |
| 6 | d | no | 1 | s[3]='t' no | 1 | yes | 2 |
| 7 | e | yes | 2 | s[4]='c' no | 2 | yes | 2 |

**Result:** `best = 2`.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n·k) | O(k) | Rescans overlapping characters; TLE at upper constraint limits |
| Sliding Window | O(n) | O(1) | Optimal; accepted solution above |

Same fixed-size sliding-window skeleton as *Maximum Average Subarray I*: maintain one running aggregate over the window (there, a sum; here, a vowel count) and repair it in O(1) per shift instead of recomputing the whole window.