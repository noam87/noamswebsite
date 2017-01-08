+++
date = "2013-10-05"
title = "Amdahl's Law"
draft = false
wikis = ["computers", "math"]
tags = ["concurrency", "featured"]
+++

> As multicore computing becomes the norm (even my phone is dual core!), it's important to understand the benefits and also the limitations of concurrency. Amdahl's Law addresses the latter.

Let's imagine a simple program. It prints "Hello World" 100 times, then quits.

Our first version of the program is written as a single sequential task: it prints one "Hello World", then another, then another, 100 times, then quits.  This program takes some unit of time, $t$ to execute.

Now say we have a dual-core machine at hand. (My phone, perhaps).

Cool, now we can spawn *two* tasks that print "Hello World" 50 times each. And, because our magical imaginary computer experiences no overhead, it takes us exactly $\frac{ t }{ 2 }$ units of time to run our second program.

So we keep adding more and more processors, until we have 100 concurrent threads printing one "Hello World" each, and our program runs 100 times faster.

At this point we stop: "Ah, the trend is clear: more processors equals more speed! No point in continuing this experiment."

**A naive (wrong) first guess:** Given $n$ processors executing a program, the maximum boost in speed is $n$. (That is, we can get our program to run $n$ times faster).

Cool! This means that, given enough processors, we could make *any* program run almost instantly. Right?

![](/img/more_cores.jpg)

([Pic original source](http://forums.pureoverclock.com/amd/21809-rumor-mill-amd-iv-x12-170-12-cores-24mb-cache-6ghz-2.html#post169754))

Of course this is not the case! Enough daydreaming. Let's figure out a more <!--more--> realistic estimate.

Let $P$ be the proportion of our program that can run in parallel. Then it follows that $1 - P$ is the proportion that cannot be broken up into independent tasks.

For example, since our program can be broken up into 100 independent tasks, then $1 - P = \frac{ 1 }{ 100 }$.

It follows that the maximum boost in speed (denoted $S(n)$) that we can expect out of assigning concurrent tasks to $n$ parallel processors can be represented by the following equation:

$$S(n) = \frac{ 1 }{ (1 - P) + \frac{ P }{ n } }$$

This is, in fact, Amdahl's equation.

Uh-oh... do you see it? As we add more and more processors to our computer, and $n \to \infty$, we are left with $ S =  \frac{ 1 }{ 1 - p }$.

What we have here is a clear case of *diminishing returns.*

How bad is it?  Let's add *one million cores* to our imaginary computer, and measure its performance at $gc = 99\%$:

![](/img/99pc.gif)

Well, for our imaginary software, 99% of which can be parallelized, we can expect a maximum boost of $ S = 100$.

What about a program with $gc = 90\%$?

![](/img/90pc.gif)

There's that same plateau again. But this time we're only seeing a maximum performance boost of $S = 10$.

By $gc = 50\% $, we're down to a program that can only be boosted to run twice as fast no matter how much parallel processing your machine is capable of!

**Final Note:** In fact, Amdahl's Law is not exclusive to concurrency, but applies to *any* speed-boosting strategy that only affects some portion of a program.
