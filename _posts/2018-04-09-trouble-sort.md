---
layout: post
title:  "Code Jam 2018: Trouble Sort"
date:   2018-04-09
description: "Bubble sort: done wrong."
permalink: trouble-sort
---

<p class="info">Google Code Jam is an annual, international programming competition. Similar to ACM/ICPC but less restrictive since anyone who registers online can compete. Also the contest is not team based. It consists of several stages until its final onsite round. The very first stage qualification round recently took place.</p>

I have checked the problems and proposed some solutions to those. 

**Disclaimer**: Proposed ideas and implementations are not official, ideal, nor coded optimally. Instead, they are only one way to tackle the problems. Feel free to contact me to feedback possible optimizations!

**Prerequisite**: This explanation assumes that you are familiar with the terms, variables and the problem itself as defined on the official Code Jam page. If you have not read the problem description, please take your time and go through all of it.

# Rudimentary Approach

Before highlighting any observations, the most rudimentary approach looks tempting:

- Run the trouble sort
- Pass over the array to detect an *anomaly*
- If there is no *anomaly* then output `OK`
- Otherwise, output index of *anomaly*

What deters one from doing this is the **constraint** on the *Hidden Test Set*: \\(3 \leq N \leq 10^5\\). We know that the average and the worst-case performance of bubble sort is \\(O(n^2)\\). See some assumptions, and deductions below:

- Assume 1 B instructions on a *regular machine* run in 1 second
- Assume that the computation power allocated for a submission on Google Code Jam judge server is in the order of a *regular machine*
- Assume trouble sort has the same average and worst-case complexity as bubble sort.


Then trouble sort requires \\(10^{10}\\) operations to execute, which would take ~10 seconds on Google Code Jam judge server (for a single large test case). If, we have more than a few such large test cases, it is nearly certain that this approach will not pass *Hidden Test Set*.

# Observations

The crucial detail in trouble sort is that:

- It compares and sorts the values at *even* indices among themselves.
- It compares and sorts the values at *odd* indices among themselves.

Thus, even though trouble sort works on a single array, it sorts two different parts of the array *independently*. It is possible that when these two arrays are interleaved they may not form a sorted array.

# Approach

We know we could perform sorting in \\(O(n \log{n})\\). Therefore, the idea is:
- Extract the *even* array from the original array and sort it in \\(O(n \log{n})\\).
- Extract the *odd* array from the original array and sort it in \\(O(n \log{n})\\).
- Make a pass through both arrays in an alternating manner and detect if there is an *anomaly*.

# Implementation

The following is a C++ implementation of the approach mentioned.

```cpp
/**
 * Ans is a wrapper class to store answers
 * Case is a wrapper class to store cases
 */

using vlong = vector<long>;

Ans solveCase(Case &cs){
    Ans ans;
    ans.position = -1; // -1 indicates 'no anomaly', assume initially

    vlong even;
    vlong odd;
    extract(cs, even, odd);

    for(int i=0; i<odd.size(); i++){ // the alternating pass
        if(odd[i]<even[i]){
            ans.position = i*2;
            break;
        }else if(i+1 < even.size() && even[i+1]<odd[i]){            
            ans.position = i*2+1;
            break;
        }
    }

    return ans;
}
```

Notice that the odd array could be 1 element shorter then the even array. When they are of equal length, we have an irregularity in the comparison pattern. See the following figure, for the demonstration of what the for loop does in this code.

<center><img src="/assets/trouble-sort-arrays.svg"></center>
<br>

The last piece of this implementation is the following straightforward function to extract the *even* and *odd* arrays.

```cpp
void extract(Case &cs, vlong& even, vlong& odd){
    for (int i = 0; i < cs.V.size(); ++i) {
        if(i%2){
            odd.push_back(cs.V[i]);
        }else{
            even.push_back(cs.V[i]);
        }
    }
    sort(even.begin(), even.end());
    sort(odd.begin(), odd.end());
}
```

# Improvement

With this implementation, we require auxiliary space and a tedious alternating pass to detect anomaly. A more elegant way of performing this would be:
- Instead of extracting *even* and *odd* arrays, use two custom iterators on original array that visits *even* and *odd* indices respectively to sort.
- By doing so, we require no auxiliary space and the anomaly detection would be a simpler procedure.
