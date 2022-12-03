---
title: Leetcode 32. Longest Valid Parentheses
date: 2022-12-03
description: "It’s not that hard by the way"
tags: [Leetcode, Kotlin, Algorithm]
thumbnail: ""
---

Given a string containing just the characters '(' and ')', return the length of the longest valid (well-formed)
parentheses substring.

Example 1:

```shell
Input: s = "(()"
Output: 2
Explanation: The longest valid parentheses substring is "()".
```

Example 2:

```shell
Input: s = ")()())"
Output: 4
Explanation: The longest valid parentheses substring is "()()".
```

Example 3:

```shell
Input: s = ""
Output: 0
```

Constraints:

- 0 <= s.length <= 3 * 104
- s[i] is '(', or ')'.

## Approach

1. Loop through the string from left to right and store the counts of both type of parentheses in two variables left and
   right
2. If left == right, it means we have valid substring.
3. We can then find if the length of current valid substring (left + right) is the maximum or not.
4. If right > left, it means we have invalid strings, and we will reset left and right to zero.
5. Repeat the steps 1-4 looping string from right to left and reset counters as soon as left > right.


### Time Complexity
Since we are looping the string twice, the time complexity will be O(n).

### Space Complexity
We are not using any data structures to store intermediate computations, hence the space complexity will be O(1).

## Code

```kotlin
fun longestValidParentheses(s: String): Int {
    var count = 0
    var left = 0
    var right = 0

    for (i in s.indices) {
        if (s[i] == '(') left++
        if (s[i] == ')') right++

        if (left == right) count = Math.max(count, left + right)

        if (right > left) {
            right = 0
            left = 0
        }
    }

    left = 0
    right = 0

    for (i in s.length - 1 downTo 0) {
        if (s[i] == '(') left++
        if (s[i] == ')') right++

        if (left == right) count = Math.max(count, left + right)

        if (left > right) {
            right = 0
            left = 0
        }
    }

    return count
}
```