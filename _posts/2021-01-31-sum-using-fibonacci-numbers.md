---
layout: post
title:  "Sum to any number using Fibonacci numbers" 
date:   2021-01-31
description: Also known as Zeckendorf's theorem.
comments: true
permalink: not-darkest-dungeon
---

This is also known as **Zeckendorf's theorem** and 
[**Wikipedia**](https://en.wikipedia.org/wiki/Zeckendorf%27s_theorem){:target="_blank"} already explains it in 
the mathematical notation but this post is for those who want a verbal explanation.

> **The Problem** <br>
> - Can you sum to an arbitrary positive integer \\(K\\) using the numbers \\(f_i\\) from the Fibonacci series? 
> - If so, can you do that using as few \\(f_i\\) as possible?
  
It turns out that the following observation answers both of these questions:

**If \\(K\\) is a Fibonacci number,** then we know that yes, we can sum to \\(K\\) using Fibonacci number \\(K\\) 
and that would be the minimal set of \\(\\{K\\}\\).

**Otherwise, \\(K\\) lands between two Fibonacci numbers,** say \\(f_i\\) and \\(f_{i+1}\\) so that \\(f_i < K < f_{i+1}\\).
Greedily, we can reduce this problem to a smaller problem of the same kind: Can we sum to \\(K-f_i\\) using the minimum number of 
Fibonacci numbers? If we would have the answer to that, then we can construct an answer to the original problem by 
simply adding \\(f_i\\) to the summation list. By induction, our problem shrinks down to the initial numbers in the Fibonacci series
\\(\\{1, 1, 2, 3, 5, ...\\}\\). It's easy to spot the minimal summation list when \\(K < 4\\). Since the base case holds 
our induction holds. 

Therefore, we can always construct a list with a minimum number of Fibonacci numbers summing to an arbitrary positive 
integer \\(K\\), by greedily adding the preceding Fibonacci number \\(f_i\\) to the list and iterating the problem with 
\\(K-f_i\\).

# The Code

An important remark is that `binary_search` in the code below returns the Fibonacci number itself (i.e. \\(f_i \leq K\\)) 
instead of its index. Also, `generate_fibonacci_numbers(k)`, creates a list with Fibonacci numbers up to \\(K\\). This 
code apparently returns the size of the summation list instead of the list itself, however, it's trivial to alter the 
code to output the actual list. 

```cpp
int sum_with_fibonacci_numbers(int k) {
    vector<int> nums = generate_fibonacci_numbers(k);
    k -= binary_search(nums, k);
    int num_count = 1;
    while (k > 0) {
        k -= binary_search(nums, k);
        num_count++;
    }
    return num_count;
}
```
