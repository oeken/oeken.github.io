---
layout: post
title:  "Code Jam 2018: Saving the Universe Again"
date:   2018-04-08
description: A greedy approach to protect earth from aliens.
permalink: saving-the-universe-again
---

<p class="info">Google Code Jam is an annual, international programming competition. Similar to ACM/ICPC but less restrictive since anyone who registers online can compete. Also the contest is not team based. It consists of several stages until its final onsite round. The very first stage qualification round recently took place.</p>

I have checked the problems and proposed some solutions to those. 

**Disclaimer**: Proposed ideas and implementations are not official, ideal, nor coded optimally. Instead, this is only one way to tackle the problem. Feel free to contact me to feedback possible optimizations!

**Prerequisite**: This explanation assumes that you are familiar with the terms, variables and the problem itself as defined on the official Code Jam [**page**](https://codejam.withgoogle.com/2018/){:target="_blank"}. If you have not read the problem description, please take your time and go through all of it.

# Observations

Naming things got pretty easy in recent years. Another title of the form: 

`<verb>ing the <object> great[optional] again!`

Important observations about this problem are:
- An exchange of adjacent letters that are both `S` is useless, since it does not reduce the total damage dealt to earth.
- Given infinite number of *hacks* (a.k.a. exchange), it is always possible move all `S` to the very beginning (left hand-side) of the sequence.
- Given a sequence, the damage dealt to earth would be minimal when all `S` are stacked to the very beginning of sequence. Giving damage of total number of `S` to earth.

# Approach

Now we know what to do to check if it is possible or `IMPOSSIBLE` to protect earth given an attack sequence:

- If number of `S` is less than or equal to `D` then *possible*
- Otherwise `IMPOSSIBLE`

Then... what should we do if it's *possible* to protect the earth? How do we find the minimal number of hacks?

We notice that a greedy algorithm could be used for finding it. Say, that the current damage dealt in the current state of the sequence is `V`, then we would like to reduce this attack damage by `GAP = V - D` in order to protect earth. If we shift the **right-most** `S` in the sequence, we are making progress at the greatest rate. Making progress at the greatest rate implies minimal number of *hacks*.

What if the right-most `S` is preceded by another `S`?

Then doing a *hack* there would be useless. In that case we would like to find the **right-most** `S` **that is preceded by a** `C` and *hack* that one. Notice that each attack in a contiguous sequence of `S` deal same damage to earth.

# Implementation

The idea looks simple enough, here is an implementation of this approach in C++

```cpp
/**
 * Ans is a wrapper class to store answers
 * Case is a wrapper class to store cases
 */

using vint = vector<int>;
using vlong = vector<long>;

Ans solveCase(Case &cs){
    Ans ans; 
    ans.count = -1; // -1 indicates IMPOSSIBLE
    // make a pass in sequence
    vint shootPos;  // stores positions of S
    vlong shootVal; // stores damages of S
    long val = 1;   // damage at the current position
    long total = 0; // stores total damage of sequence
    for(int i=0; i<cs.P.size(); i++){
        if(cs.P[i]=='S'){ // we have an S at this position
            shootPos.push_back(i);
            shootVal.push_back(val);
            total += val;
        }else{ // we have a C at this position
            val *= 2;
        }
    }

    // check if possible to protect earth
    if(shootPos.size() <= cs.D){ 
        ans.count = countHacks(shootPos, shootVal, total-cs.D);
    }
    return ans;
}
```

The following function is the critical one for this problem. It is possible to implement this in various ways.
A sub-procedure of this function is to find the **best fit** `S` in the given the set of `S` with two properties `Position` and `Damage`. In such scenarios, one typically uses an Heap with custom comparator. Here instead, we chose to search it linearly, as the constraints were relaxed and implementation was simpler.

```cpp
int countHacks(vint &pos, vlong& val, long gap){
    int count = 0;
    while(gap>0){
        // search linearly to find hack index
        int hackIndex = -1;
        for(int i=val.size()-1; i>=0; i--){
            if(i-1<0 || pos[i]-pos[i-1]>1){ // found
                hackIndex = i;
                break;
            }
        }
        // hack it
        pos[hackIndex] -= 1; 	// shift it to left
        val[hackIndex] /= 2; 	// halve the damage
        gap -= val[hackIndex]; 	// update gap
        count++; 		// increment hack count
    }
    return count;
}
```

