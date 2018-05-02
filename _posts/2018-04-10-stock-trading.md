---
layout: post
title:  "Stock trading using at most T transactions"
date:   2018-05-02
comments: true
description: A dynamic programming puzzle.
permalink: stock-trading
---

**Prerequisite**: This explanation assumes that you are familiar with the concept of [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming){:target="_blank"}.


Say that we are a bunch of smart people who want to make money by stock trading. We cannot predict the future, but at least we would like to know how much money we could have made maximally if we knew the stock prices in future days. In the minimalistic world we live in, there is only one company offered to the public. We are given an array of integers `A` representing the stock prices of the company every day. In addition, we are given a number `T` representing the maximum number of transactions we could perform. A transaction is buying a stock and selling it later. Let us assume one more rule: we cannot do transactions concurrently. I.e. if we bought a stock on a day we have to sell it before we would like to buy another one. Imagine as if only a single stock exists. But also we are not concerned that whether this single stock is unavailable or not when we do not possess it. It is reserved for the interaction of ours only.

Let's see some example cases to get a better grasp of what we are dealing with.  

#### Example Case 1

`A`: \\( [3,3,5,0,0,3,1,4] \\)  
`T`: \\(2\\)  

In this particular instance of this problem, the optimal strategy would be:
- Buy on day 4
- Sell on day 6 (profit: 3)
- Buy on day 7
- Buy on day 8 (profit: 3)
Thus the total profit would be 6. 

#### Example Case 2

`A`: \\( [7,6,4,3,1] \\)  
`T`: \\(2\\)  

In this instance, there is no way to make a profit by performing a transaction. So we rather do not bother at all and the total profit would be 0. Recall that we are **not** obliged to use `T` transactions. Rather, we could use at most `T` transactions.


# Approach

We want to come up with a computer program that figures out the optimal strategy for us. For simplicity reasons our program is going to output only the *maximum profit* rather than the history of transactions. We follow a dynamic programming approach. Let's begin by defining the Bellman equation:

Let \\(f[i][t]\\) be the maximal profit that can be made using the first \\(i\\) items of the array by using at most \\(t\\) transactions.  

According to this definition we observe that the base case is:
- \\(f[i][0] = 0\\), for any \\(i \geq 0\\)
- \\(f[0][t] = 0\\), for any \\(t \geq 0\\)

Assume that array \\(A\\) is 0-based and size of \\(N\\). We define the recursive case as the following:

$$
f[i][t] = max
\left \{
\begin{align} 	
	&f[i-1][t],\\ 
	&f[i-1][t-1] + (A[i-1] - A[i-2]),\\
	&f[i-2][t-1] + (A[i-1] - A[i-3]),\\
	&f[i-3][t-1] + (A[i-1] - A[i-4]),\\
	&.\\
	&.\\
	&f[1][t-1] + (A[i-1] - A[0])\\
\end{align}
\right \}
$$

**Explanation**: Consider the problem \\(f[i][t]\\), that is when we are taking first \\(i\\) stock prices into account and there are \\(t\\) transactions available to us. For this problem there exists an optimal list of transactions which maximizes profit. Consider the very last transaction in this optimal list:
- It takes place after all others in the list.
- We would like to figure out when this last transaction takes place (time it's bought and time it's sold), but it is not obvious.
- If we make a guess for when it takes place (e.g. bought at \\((i-3)\\) and sold at \\((i-1)\\)) then by solving a smaller version of this problem (a sub-problem) we can give an answer to the current problem.
	- In that sub-problem, we would have 1 less transaction available, and would consider the array of prices that come before the purchase we guessed.
- Assume that we already have the answers to the sub-problems.
- So the idea is to consider all such guesses and take the one that yields the maximum profit.

To concretize let us explain what each line in the equation above means. Recall, we are considering the problem \\(f[i][t]\\).

- \\(f[i-1][t]\\), no action at time \\(i-1\\)
- \\(f[i-1][t-1] + (A[i-1] - A[i-2])\\), sell your stock at time \\(i-1\\) and guess that it was bought at time \\(i-2\\)
- \\(f[i-2][t-1] + (A[i-1] - A[i-3])\\), sell your stock at time \\(i-1\\) and guess that it was bought at time \\(i-3\\)
- \\(f[i-3][t-1] + (A[i-1] - A[i-4])\\), sell your stock at time \\(i-1\\) and guess that it was bought at time \\(i-4\\)
- So on until to the last possibility...
- \\(f[1][t-1] + (A[i-1] - A[0])\\), sell your stock at time \\(i-1\\) and guess that it was bought at time \\(0\\)

Notice that with this recursive definition (thanks to the first equation i.e. \\(f[i-1][t]\\)) we do not need to search for selling time of the last stock. Algorithm intrinsically considers all *selling time choices* by being able to skip a timestep.

See the image below which demonstrates the dynamic programming table and the dependencies originating from our recursive definition. 
- The purple box is the current problem \\(f[i][t]\\). 
- Blue boxes are its dependencies. 
	- Boxes with red spots are the sub-problems when we consider selling at time \\(i-1\\).
	- The one with the green spot is the sub-problem when *no-action* is considered.


<center><img width="500px" style="margin-top:20px;margin-bottom:30px;" src="/assets/dp-table.svg"></center>

# Implementation Trick

It looks all good except for one fatal failure. Dynamic programming is useful only when constructing a solution to a problem from the solutions of sub-problems is easy/efficient. In the recursive definition we gave above, for each cell of the table we need to iterate over solutions to many other sub-problems, rendering this approach \\(O(N^2 T)\\) in terms of time and \\(O(N T)\\) in terms of space. Where \\(N\\) is the size of the array and \\(T\\) is the number of transactions allowed.

**Observation**: Luckily with an implementation trick we can reduce the time complexity to \\(O(N T)\\). Below, compare what is needed to compute \\(f[i][t]\\) and \\(f[i+1][t]\\) and see if we can simplify computation of \\(f[i+1][t]\\).

$$
\scriptsize
f[i][t] = max
\left \{
\begin{matrix} 	
f[i-1][t],\\ 
f[i-1][t-1] + (A[i-1] - A[i-2]),\\
f[i-2][t-1] + (A[i-1] - A[i-3]),\\
f[i-3][t-1] + (A[i-1] - A[i-4]),\\
.\\
.\\
f[1][t-1] + (A[i-1] - A[0])\\
\end{matrix}
\right \}
\hspace{5mm}

f[i+1][t] = max
\left \{
\begin{matrix} 	
f[i][t],\\ 
f[i][t-1] + (A[i] - A[i-1]),\\
f[i-1][t-1] + (A[i] - A[i-2]),\\
f[i-2][t-1] + (A[i] - A[i-3]),\\
.\\
.\\
f[1][t-1] + (A[i] - A[0])\\
\end{matrix}
\right \}
$$

Noticed a pattern? 

$$
\scriptsize
max
\left \{
\begin{matrix} 	
f[i-1][t-1] + (A[i-1] - A[i-2]),\\
f[i-2][t-1] + (A[i-1] - A[i-3]),\\
f[i-3][t-1] + (A[i-1] - A[i-4]),\\
.\\
.\\
f[1][t-1] + (A[i-1] - A[0])\\
\end{matrix}
\right \}

+(A[i]-A[i-1])

= 

\left \{
\begin{matrix}
f[i-1][t-1] + (A[i] - A[i-2]),\\
f[i-2][t-1] + (A[i] - A[i-3]),\\
f[i-3][t-1] + (A[i] - A[i-4]),\\
.\\
.\\
f[1][t-1] + (A[i] - A[0])\\
\end{matrix}
\right \}
$$

From the expression above we can draw the following observation:  
If we keep track of maximum of last \\(i-1\\) elements used in the computation of \\(f[i][t]\\) (let us call it `runningMax`) then we can construct an answer to \\(f[i+1][t]\\) in \\(O(1)\\) by:
- Converting `runningMax` by adding `(A[i]-A[i-1])`
- Computing `max(runningMax, f[i][t], f[i][t-1] + (A[i]-A[i-1]))` as the answer
	- The latter two are the first two dependencies needed for the computation of \\(f[i+1][t]\\)

For the computation of following problems (e.g. \\(f[i+2][t]\\), \\(f[i+3][t]\\), ...) we also need to keep `runningMax` updated by doing the following:  
`runningMax = max(runningMax, f[i][t-1] + (A[i]-A[i-1]))`

# Implementation

Below is the implementation in C++. 

```cpp
using vint = vector<int>;
using vvint = vector<vint>;

int maxProfit(vint &A, int T){
    if(A.empty()) return 0;
    int N = A.size();
    vvint f(N+1, vint(T+1,0));
    for (int t=1; t<=T; ++t) {
        int runningMax = 0;
        f[1][t] = 0;
        for(int i=1; i<N; i++){
	    int change = A[i]-A[i-1];
            runningMax += change; // convert running max
            runningMax = max(runningMax, f[i][t-1]+change); // update running max
            f[i+1][t] = max(f[i][t], runningMax); // the answer
        }
    }
    return f[N][T];
}
```

# Remarks

This is one of the not very obvious dynamic programming problems and in this particular formulation the implementation trick plays a very important part since otherwise time complexity is simply too expensive.