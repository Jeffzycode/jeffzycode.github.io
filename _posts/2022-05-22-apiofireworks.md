---
layout: post
title:  "APIO 2016 B Fireworks: Slope Trick on a Tree"
date:   2022-05-22 00:00:00 -0400
categories: APIO SlopeTrick
---

Rather than writing all of my comments in my notes, I’ve decided to write them here instead. This is the
first 40p problem that I’ve attempted, and I will try not to read any editorials when solving this problem.
It's highly recommended to read the problem statement before reading this article in order to understand it
well.

[Link to the Problem](https://dmoj.ca/problem/apio16p2)

At first glance, this problem looks like it will be Tree DP. I know that I will need to use the slope trick as
well, which seems to be heavily implied by the problem statement anyways, so let's first attempt to lay
down our plan for how to tackle each transition.

Let $$f_k$$ denote the function which represents the minimum cost of resizing the fuses in the subtree rooted at $$k$$ such that the all of the paths from to each root in the subtree is equal to $$x$$. Since switch no. 1 is the root of the tree, if we can derive $$f_1(x)$$, we can query it for our solution.

Now, for our transitions. We know if the child of node $$k$$ is a firework, and the fuse between node $$k$$ and
the firework is of length $$m$$, then $$f_k(x)=\vert x-m\vert$$. It would be logical for to $$f(x)$$ always be convex,
but let's see how we can merge our functions.

Without loss of generality, let node $$j$$ be a child of node $$k$$. Furthermore, let $$L(j,k)$$ denote the length of the
fuse between $$j$$ and $$k$$. We know that we have to somehow merge $$f_j(x)$$ and $$\vert x-L(j,k)\vert$$. This is probably
where the most difficult part of the problem comes in.

Let's use desmos to help us visualize. For exemplary purposes, consider the following part of the sample
case:

<div style="text-align:center">
  <img src="/assets/APIOFireworks/APIOF1.png">
</div>

We can easily solve by hand to see that $$f_3(x)=\vert x-3\vert+\vert x-2\vert+\vert x-3\vert$$. Let us define $$g_{2,3}(x)=\vert x-5\vert$$ and attempt to merge $$f_3(x)$$ and $$g_{2,3}(x)$$ into some function $$h_{2,3}(x)$$. At face-value, lets try and define $$h_{p,q}(x)$$, where $$q$$ is $$p$$’s child node. $$h_{p,q}(x)$$ denotes the minimum cost to resize the fuses in the subtree rooted at $$q$$ and also the fuse connecting $$p$$ and $$q$$ such that the distance between node $$p$$ to any firework in the subtree rooted at $$q$$ is equal to $$x$$. Using this definition, we can conjecture that $$f_{k}(x)=\sum_{i \in \mathrm{child}(k)} h_{k,i}(x)$$ assuming that $$h_{p,q}(x)$$ is a convex function.

Coming back to the case of $$h_{2,3}(x)$$, we know that we can either resize some fuses in the subtree rooted at $$3$$, or we can resize the fuse between $$2$$ and $$3$$, the cost of doing either being $$f_3(x)$$ and $$g_{2,3}(x)$$. Hence, $$h_{2,3}(x)=\min(f_3(m)+g_{2,3}(n) \forall \{m+n=x\})$$. Since no fuse can be resized into the negatives, our domain is bounded to non-negative numbers. Hence, $$h_{2,3}(x)=\min(f_{3}(m)+g_{2,3}(x-m)\{m\in \mathbb{Z}_{\ge 0},0\le m\le x\})$$. Calculating this function is very slow, and implementing this solution naively can solve the second subtask only.

Let's [graph](https://www.desmos.com/calculator/665z50kxmd) the function to look for some patterns. Try playing around with the slider for $$z$$. We can make several observations outright:
* $$h_{2,3}(x)$$ is minimum when $$x=8$$. By extension, we recognize that $$h_{p,q}(x)$$ is minimum when $$x=\min(x)$$ 𝗈𝖿 $$f_q(x)+L(p,q)$$. The proof is as follows: the magnitude of the slope of $$g_{p,q}(x)$$ is always $$1$$, while the magnitude of the slope of $$f_{q}(x)\ge 1$$ outside the minimum zone, and the minima of $$g_{p,q}(x)$$ is always $$0$$ while the minima of $$f_{q}(x)\ge 0$$, hence the function is always minimum when $$g_{p,q}(x)=0$$. 
* $$h_{2,3}(x)$$ has a slope of $$1$$ when $$x>8$$. By extension, we recognize that $$h_{p,q}(x)$$ has a slope of $$1$$ when $$x>\min(x)$$ 𝗈𝖿 $$f_{q}(x)+L(p,q)$$. The proof is as follows: the slope of $$f_{q}(x) \ge 1$$ when $$x$$ is greater than the minimum of $$f_q$$, while the slope of $$g_{p,q}(x)=1$$ when $$x$$ is greater than $$L(p,q)$$. Hence, in this range, it’s always more or just as optimal to increase the length of fuse $$pq$$ to account for the difference between $$x$$ and the (largest) minima of $$f_q$$, rather than adjusting any fuses in the subtree rooted at $$q$$.
* $$h_{2,3}(x)$$ has a slope of $$-1$$ when $$3<x<8$$. By extension, we recognize that $$h_{p,q}(x)$$ has a slope of $$-1$$ when $$x < \min(x)$$ 𝗈𝖿 $$f_{q}(x)<x<\min(x)$$ 𝗈𝖿 $$f_q(x)+L(p,q)$$. The proof essentially emulates the prior point: the slope of $$f_{q}(x)\le -1$$ when $$x<\min(x)$$ 𝗈𝖿 $$f_{q}(x)$$, while the slope of $$g_{p,q}(x)=-1$$ when $$x<L(p,q)$$. Hence, it’s always more or just as optimal to decrease the length of fuse $$pq$$ to account for the difference between $$x$$ and the (smallest) minima of $$f_q$$, rather than adjusting any fuses in the subtree rooted at $$q$$.
* It is slightly harder to see on the graph, but the slope of $$h_{2,3}(x)$$ is equal to the slope of $$f_3(x)$$ when $$x<3$$. By extension, the slope of $$h_{p,q}(x)$$ is equal to the slope of $$f_{q}(x)$$ when $$x<\min(x)$$ 𝗈𝖿 $$f_{q}(x)$$. The proof more complicated, but a non-rigorous version is an extension of the previous proof: since we can no longer decrease the length of fuse $$pq$$, we are forced to adjust the fuses in the subtree rooted at $$q$$. The rigorous version is left as an exercise to the reader.

Using these observations, we can efficiently calculate $$h_{p,q}(x)$$ from $$g_{p,q}(x)$$ and $$f_{q}(x)$$ using two priority queues, denoting the left side and right side of the slope trick function (see the slope trick CF article). At the end of the merging process, the right priority queue should only consist of a single number, since the slope of $$h_{p,q}(x)=1$$ when $$x>\min(x)$$ 𝗈𝖿 $$f_{q}(x)+L(p,q)$$.
1. Copy everything in the left priority queue from $$f_q(x)$$.
2. Pop the rightmost element from the copied left priority queue.
3. Add the smallest $$\min(x)$$ 𝗈𝖿 $$f_{q}(x)+L(p,q)$$ to the left priority queue. Effectively, take the frontmost element of $$f_q(x)$$’s left priority queue, add $$L(p,q)$$ and push it into $$h_{p,q}(x)$$’s left priority queue.
4. Add the largest $$\min(x)$$ 𝗈𝖿 $$f_{q}(x)+L(p,q)$$ to the right priority queue. Effectively, take the frontmost element of $$f_q(x)$$’s right priority queue, add $$L(p,q)$$ and push it into $$h_{p,q}(x)$$’s right priority queue.
5. Use the function with slope 1 passing through this point as a reference, or record the value of $$h_{p,q}(x)$$ for this point, which is precisely $$\min(f_q(x))$$.

$$\begin{aligned}
  y=x+b\\
  h_{p,q}(x)=x+b\\
  b=h_{p,q}(x)-x
\end{aligned}$$

Since $$h_{p,q}(x)$$ inherits values from two convex functions, $$h_{p,q}(x)$$ is also convex. Likewise, $$f_k(x)=\sum_{i \in \mathrm{child}(k)} h_{k,i}(x)$$ is convex. Lastly, we must efficiently merge two values of $$h_{p,q}(x)$$, while maintaining the minimum value. Having two priority queues complicates things, and we only store one value in the right priority queue anyways for $$h_{p,q}(x)$$, so we can modify step $$1$$ to pruning everything from $$f_q(x)$$'s priority queue until the slope is equal to $$0$$. In a worst case scenario, we have a binary tree: in this case, we need $$\mathcal{O}\left(\left(\frac{N}{2}+\frac{N}{4}2+\frac{N}{8}4+\ldots+\frac{N}{\log_2(N)}(\log_{2}(N)-1)\right)\log_{2}(N)\right)=\mathcal{O}(N\log^2(N))$$ merges. In a binary tree, $$M=2N = 2\times 10^5$$ due to the constraint that $$M+N \le 3 \times 10^5$$, which is enough to pass. Note that when creating $$h_{p,q}(x)$$, we only ever pop any element out once, so the total complexity of creating $$h_{p,q}(x)$$ is $$\mathcal{O}(N)$$, which only increases our final time complexity by a constant factor. Please note that the final complexity is likely $$\mathcal{O}((N+M)\log^2(N+M))$$ with a very good practical time complexity.

Implementing this algorithm doesn’t seem annoying at all. Let's attempt it.

<div style="text-align:center">
  <img src="/assets/APIOFireworks/APIOF2.png">
</div>

<p style="text-align:center"><i>Totally not second try because I forgot to set some variables to long long</i></p>

![](/assets/APIOFireworks/APIOF3.gif)

<p style="text-align:center"><i>Surely summer wasn't over...</i></p>

----

I've transcribed the old blog post from two years ago almost verbatim (looking back, I made some questionable choices with math notation...). A minor clarification: when I wrote $\min(x)$ of $f(x)$, I believe I meant $\argmin f(x)$.

