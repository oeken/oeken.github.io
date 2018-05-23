---
layout: post
title:  "Cartooning Images"
date:   2018-05-20
description: A variational approach to cartoonify RGB images.
comments: true
permalink: image-cartooning
fig-size: 280
---

In this post, I will demonstrate a simple application of variational methods in computer vision. We receive an RGB image of any size as input and want to give it a cartoonish looking style.

<center>
  <div style="margin-left:-20px;">
    <img width="{{ page.fig-size }}px" src="/assets/image-cartooning/original.png" style="margin-top:20px;margin-bottom:30px;margin-right:10px">

    <img width="{{ page.fig-size }}px" src="/assets/image-cartooning/final.png" style="margin-top:20px;margin-bottom:30px;margin-right:10px">
  </div>  
</center>

**Variational methods?** In these approaches, one typically specifies a probability distribution over the hidden variables that are conditioned upon observed variables and optionally a prior distribution on the hidden variables. This is the model definition phase, then one either performs an inference to uncover properties of the distribution or seeks a particular solution. A solution is just an instantiation of hidden variables, i.e. an assignment of values. Often, one looks for the mode of the distribution. A typical workflow in this approach is as follows:
1. Specify a probabilistic model that captures the problem well
2. Convert it to an error/energy function
3. Optimize the error function

> Throughout this post we use the following notation
> - \\(A \in \mathbb{R}^{kxn}\\)
> - \\(A_{i:}\\) is \\(i\\)th row of \\(A\\)
> - \\(A_{:i}\\) is \\(i\\)th column of \\(A\\)

# Model

For our problem, we will follow the above-mentioned steps as well. In fact, we are going to skip the first step and directly specify a convex error function and optimize it using projected gradient descent. In our case, the observed variables are the RGB values of the pixels and the hidden variables are the cartoonified RGB values of the pixels. Typically, error functions consist of multiple terms enforcing different characteristics for the hidden variables. For this problem, we will use two terms, a **data term** and a **regularizer term**.

$$
E =  E_{Data} + E_{Reg}
$$

#### Data Term

This term usually measures a dissimilarity between the input/observed-variables and the output/hidden-variables, thus enforcing a similarity between. In our case, it implies that *the cartoonified image should to look similar to the input image*. We define it as the following:

$$
\begin{align}
E_{Data} &=  tr(U^T F) \\
         &=  u_{11}f_{11} + u_{12}f_{12} + ... + u_{kn}f_{kn}
\end{align}

$$

> - \\(tr(.)\\) is trace operator
> - \\(U \in \mathbb{R}^{k x n}\\)
> - \\(F \in \mathbb{R}^{k x n}\\)
> - \\(n\\) is the number of pixels in input image

What do \\(U\\), \\(F\\) and \\(k\\) represent?

According to our formulation, every pixel in the cartoonified image is one of \\(k\\) colors. Here \\(k\\) is a rather small number (e.g. 5) because in general cartoon images contain only a handful of colors. Let \\(C\\) be set of cartoon colors. What colors are reasonable to have in C? There is no absolute way to answer this, though it makes sense to put some *dominant colors from the input image*. We will get back to this later.

**\\(F\\) Matrix**

Once we decide on \\(C\\), we can construct the matrix \\(F\\) where \\(F_{ij}\\) measures dissimilarity between the color of the \\(j\\)th pixel and the \\(i\\)th color in \\(C\\). For a demonstration let's say we picked \\(C = \\{red, green, blue, white, black\\}\\)  and constructed \\(F\\) accordingly. The first row of F will be a measure of distance of the original image from the color red, the second row of F for green, so on and so forth. See the following figure for a visualization of each row of F for this situation.

<center>
  <div style="margin-top:20px;margin-bottom:10px;">

    <figure style="display:inline-block;">
      <img display="block" width="{{ page.fig-size | divided_by:1.5 }}px" src="/assets/image-cartooning/red.svg">    
      <figcaption><i>Distances to Red</i></figcaption>
    </figure>

    <figure style="display:inline-block;">
      <img width="{{ page.fig-size | divided_by:1.5 }}px" src="/assets/image-cartooning/green.svg">
      <figcaption><i>Distances to Green</i></figcaption>
    </figure>

    <figure style="display:inline-block;">
      <img width="{{ page.fig-size | divided_by:1.5 }}px" src="/assets/image-cartooning/blue.svg">
      <figcaption><i>Distances to Blue</i></figcaption>
    </figure>

    <figure style="display:inline-block;">
      <img width="{{ page.fig-size | divided_by:1.5 }}px" src="/assets/image-cartooning/white.svg">
      <figcaption><i>Distances to White</i></figcaption>
    </figure>

    <figure style="display:inline-block;">
      <img width="{{ page.fig-size | divided_by:1.5 }}px" src="/assets/image-cartooning/black.svg">
      <figcaption><i>Distances to Black</i></figcaption>
    </figure>
  </div>  
</center>


**\\(U\\) Matrix**

In our problem, U matrix is the hidden variables we are going to optimize for. It is not a cartoon image though, but a voting table. In our formulation, every pixel will vote for colors in C to be converted to. For instance, if C contains 3 colors and pixel 1 votes:

- 20% for \\(c_1\\) (color 1 in C)
- 70% for \\(c_2\\) (color 2 in C)
- 10% for \\(c_3\\) (color 3 in C)  

then we pick \\(C_2\\) to be the color for pixel 1 in the cartoonified image. Therefore, once we decide on U, we construct the cartoonified image with the "elected" colors for each pixel. Note that we have the following constraints for U:

> - \\(0 \leq U_{ij} \leq 1\\),  for \\(1 \leq i \leq k\\) and \\(1 \leq j \leq n\\)
> - \\( \|\| U_{:i} \|\|_1 = 1\\),  for \\(1 \leq i \leq n\\)
> 
> Where \\(\|\|.\|\|\\) is the L1-norm and \\( U_{:i} \\) is the i'th column of U.


#### Regularizing Term

This term usually measures the variance or extremity of output/hidden-variables, thus enforcing a smoothness. In our case, it implies that *the cartoonified image should consist of large intact regions*. We define it as the following:

$$
E_{Reg} =  \frac{\alpha}{2} \sum_{i=1}^{k} \| DU_{i:} \|^2
$$

> - \\(\|\|.\|\|\\) is the L2-norm operator
> - \\(D \in \mathbb{R}^{2n x n}\\) is a gradient computing matrix both in x-direction and y-direction
> - \\(\alpha\\) is a hyper-parameter adjusting the relative importance of smoothness

Construction of matrix D is beyond the scope of this post, therefore, we will assume that it is available to us.


# Approach

$$
E = tr(U^T F) + \frac{\alpha}{2} \sum_{i=1}^{k} \| DU_{i:} \|^2_2
$$

We will optimize the error function using gradient descent. The catch here is that we do not want to violate the constraints laid out for columns of U. They must stay in the set of k-dimensional [**simplex**](https://en.wikipedia.org/wiki/Simplex){:target="_blank"}. Therefore after update step, we use a sub-procedure described in [1] to project each row back to k-dimensional simplex. The details of this algorithm are again beyond the intention of this post.


#### Gradient Calculation

**Data term:** The gradient due to this term is very straightforward:

$$
E_{Data} = u_{11}f_{11} + u_{12}f_{12} + ... + u_{kn}f_{kn}
$$

$$
\frac{dE_{Data}}{dU} = \begin{bmatrix} 
                          \frac{\partial E_{Data}}{\partial u_{11}} & \frac{\partial E_{Data}}{\partial u_{12}} & \dots \\
                          \vdots & \ddots & \\
                          \frac{\partial E_{Data}}{\partial u_{k1}} &   & \frac{\partial E_{Data}}{\partial u_{kn}}
                        \end{bmatrix}

                      = \begin{bmatrix} 
                          f_{11} & f_{12} & \dots \\
                          \vdots & \ddots & \\
                          f_{k1} &   & f_{kn}
                        \end{bmatrix}

                      = F
$$

**Regularizer term:** Before computing this, let's declare two well known derivating rules when we deal with vectors and matrices.

| Variable | Gradient |
| -------------------------- |---------------------------------------------------|
| \\(x \in \mathbb{R}^{n}\\) | \\(\frac{d (\|\|x\|\|)}{d x} = \frac{x^T}{\|\|x\|\|}  \\) |
| \\(x \in \mathbb{R}^{n}\\), \\(A \in \mathbb{R}^{kxn}\\) | \\(\frac{d Ax}{d x} = A  \\)                       |

Let's unfold the summation in the regularizer term:

$$
E_{Reg} = \frac{\alpha}{2} [\|DU_{:1}\|^2 + \|DU_{:2}\|^2 + ... + \|DU_{:k}\|^2]
$$

We know that \\(\frac{d E_{Reg}}{d U} \in \mathbb{R}^{kxn} \\). Consider the first term in this series and notice that it is contributing only to the first row of \\(\frac{d E_{Reg}}{d U}\\). The same is true for other terms, every term is contributing only to their correspondent row. Therefore we can compute each row independently, for example, consider the gradient of the first term.

$$
D \in \mathbb{R}^{2nxn} \text{ and } U_{:i} \in \mathbb{R}^{n} \\

\begin{align}
\frac{d (\frac{\alpha}{2} \|DU_{:1}\|^2)}{dU_{:1}} &=  \alpha \|DU_{:1}\| \frac{d(\|DU_{:1}|\|)}{U_{:1}} \\

\frac{d(\|DU_{:1}|\|)}{U_{:1}} &= \frac{d (\|DU_{:1}\|)}{d (DU_{:1})}  \frac{d (DU_{:1})}{d U_{:1}} \textit{ (chain rule)} \\

                               &= \frac{ (DU_{:1})^T }{\|DU_{:1}\|}  D \textit{ (plug-in)} \\

\frac{d (\frac{\alpha}{2} \|DU_{:1}\|^2)}{dU_{:1}} &=  \alpha \|DU_{:1}\| \frac{ (DU_{:1})^T }{\|DU_{:1}\|}  D \\
                                                   &= \alpha U_{:1}^T D^T D  

 
\end{align}
$$

Applying same operations to the other terms, we get the following equality:

$$
\frac{dE_{Reg}}{dU} = \alpha \begin{bmatrix} 
                                U_{1:}^T D^T D\\
                                U_{2:}^T D^T D\\
                                \vdots\\
                                U_{k:}^T D^T D\\
                               \end{bmatrix} 
$$

Combining the gradient of the data-term and the of regularizer term, we get the complete gradient.

$$
 \frac{dE}{dU} = F + \alpha \begin{bmatrix} 
    U_{1:}^T D^T D\\
    U_{2:}^T D^T D\\
    \vdots\\
    U_{k:}^T D^T D\\
 \end{bmatrix}
$$

The following code computes the gradient as described:

```matlab
function grad = compute_gradient(U,F,D,alpha,k,n)
    grad1 = F;    
    grad2 = zeros(k,n);
    for i=1:k
        Du = D*U(i,:)';        
        grad2(i,:) = Du' * D;
    end    
    grad = grad1 + alpha * grad2;
end

```


# Implementation

There is one thing that we did not elaborate on: selection of color set. How do we decide which colors to use in the cartoon image?
Again there is no single answer to this. Some approaches could be:

1. Hand-pick C
2. Cluster the colors in the image and pick k of them

In my implementation, I picked C as the cluster centers resulting from the output of k-means algorithm on image colors. The colors are as following when k = 5

<center>
  <div style="margin-top:20px;margin-bottom:10px;">
    <figure style="display:inline-block;">
      <img display="block" width="{{ page.fig-size | times:2 }}px" src="/assets/image-cartooning/colors.png">    
    </figure>
  </div>  
</center>


The MATLAB script to perform projected gradient descent is as follows:

```matlab
% input
I = imread(img_path);

% init derivators
Dx = get_derivator_x(h,w);
Dy = get_derivator_y(h,w);
D = [Dx; Dy];

% parameters
alpha = 0.5;
eta = 0.2; % step size
tolerance = 1e-5; 
max_iter = 200;

% init problem
F = build_F(I,C,n,k);
U = random_init_u(k,n);

% update until convergence
for i=1:max_iter
    U_prev = U;    
    grad = compute_gradient(U,F,D,alpha,k,n);
    U = U - eta * grad;
    U = project_to_simplex(U);
            
    diff = sum(abs(U(:) - U_prev(:))) / (h * w * k);    
    if diff<tolerance
        break;
    end  
end

% build image from U
Ic = build_image(U,C,h,w);
```


See an example evolution of the dog image starting from a random initialization:

<center>
  <div style="margin-top:20px;margin-bottom:10px;">
    <figure style="display:inline-block;">
      <img display="block" width="{{ page.fig-size }}px" src="/assets/image-cartooning/anim.gif">    
    </figure>    
  </div>  
</center>

Results of some experiments with different selections of hyper-parameters:

<center>
  <div style="margin-top:20px;margin-bottom:30px;">
    <figure style="display:inline-block;">
      <img display="block" width="{{ page.fig-size | divided_by:1.2 }}px" src="/assets/image-cartooning/a-0.0-k-05.png">    
      <figcaption><i>Alpha: 0.0, k: 5</i></figcaption>
    </figure>
    
    <figure style="display:inline-block;">
      <img display="block" width="{{ page.fig-size | divided_by:1.2 }}px" src="/assets/image-cartooning/a-1.0-k-05.png">    
      <figcaption><i>Alpha: 1.0, k: 5</i></figcaption>
    </figure>

    <figure style="display:inline-block;">
      <img display="block" width="{{ page.fig-size | divided_by:1.2 }}px" src="/assets/image-cartooning/a-0.5-k-10.png">    
      <figcaption><i>Alpha: 0.5, k: 10</i></figcaption>
    </figure>  
  </div>  
</center>

# References

[1] Chen, Yunmei, and Xiaojing Ye. "Projection onto a simplex." arXiv preprint arXiv:1101.6081 (2011).

