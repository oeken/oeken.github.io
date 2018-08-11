---
layout: post
title:  "Prototyping with Autograd"
date:   2018-07-29
description: How do I quickly try out my ML model?
comments: true
permalink: prototyping-with-autograd
fig-size: 280
---

<!-- Next, we need to seek the ideal configuration of the model parameters that minimizes the loss specified. If we are lucky and the model/loss is simple enough, we can come up with 
 -->

In machine learning, upon model selection one often needs to engineer a domain specific loss function which penalizies undesirable outputs of the model. Finding the optimal configuration of the model parameters (**\\(w^*\\)**) with respect to the loss is called optimization/training. Some of the prominent optimization techniques are :

- Closed form solution
	- When we are able to express **\\(w^*\\)** in terms of other known variables and compute easily. **Example:** Linear regression
- Evolutionary optimization
	- When we combine and mutate parameters of multiple models to synthesize a better performing model. **Example:** Game playing ANNs [*](http://blog.otoro.net/2017/10/29/visual-evolution-strategies){:target="_blank"}.
- First order methods
	- When we iteratively update the model parameters using the first-derivatives (a.k.a. gradients). **Example:** DNNs trained with stochastic gradient-descent.
- Second order methods
	- When we iteratively update the model parameters using the second-derivatives. **Example:** Levenberg-Marquardt alogirthm on non-linear least squares problems.

Among these, in practice **First order methods** are far more dominant than others in the supervised and unsupervised settings. It involves computation of gradients. You could notice that if we choose to analytically derive the gradients a few serious problems arise:

- Analytic derivation is time consuming and tedious especially when the models get complicated
- We would need to re-calculate gradient function every time we alter the model or the loss function

There are several ways to delegate this work to computer:
- *[Symbolic Differentiation](https://www.wikiwand.com/en/Computer_algebra){:target="_blank"}*
- *[Numerical Differentiation](https://www.wikiwand.com/en/Numerical_differentiation){:target="_blank"}*
- *[Automatic Differentiation](https://www.wikiwand.com/en/Automatic_differentiation){:target="_blank"}*

Although in terms of space requirements automatic differentiation (AD) is relatively costly, efficiency-wise it's the best candidate we have. Advancements in the memory capacities of GPUs in the last decade allowed us to use AD for the gradient computation and consequently for the training of complicated models. The focus of this post is not on details of AD and a comprehensive guide on the topic [1] can be found for the interested people.

Since then, man


# AutoGrad Template

# Case: Logistic Regression

# Case: Soft SVM

# Remarks
takeaway message: 


# References

[1] Baydin, Atilim Gunes, et al. "Automatic differentiation in machine learning: a survey." Journal of Machine Learning Research 18.153 (2017): 1-153.


<!-- 
- Briefly tell: modeling -> engineering a loss function -> optimization -> closed form solution, gradient-based optimization, evolution based optimization
- We'll consider gradient-based 
- Tell about backprop / autodifferentiation

- How to practice?
- Use deep learning / scientific computation frameworks
- A very lightweight, rather independent (few dependencies) and a flexible one is autograd

- We show a coding template and demonstrate how quickly we can prototype a simple ML model: logit regr.
- Briefly tell about how it works?

- Do sanity check on synthetic data
- Briefly tell about the dataset we gathered
- Briefly explain the template

- Briefly explain processing on dataset
- Show some logistic regr. results

- To demonstrate it's easy to experiment convert the model into a 2layer nn
- Show 2 layer nn results
 -->