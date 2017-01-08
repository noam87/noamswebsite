+++
date = "2013-04-16"
title = "Rice's Theorem"
draft = false
wikis = ["computers", "math"]
tags = ["computation"]
+++

Rice's theorem can be stated thus:

> Every non-trivial semantic property of a program is undecidable.

Before we prove the theorem, let's break down that statement:

---

### "Semantic Property"

A ssemantic property is a property of the language, *not the machine that is
computing it*. For example, this is a semantic property of a language:

* All strings in language $L$ are of the form $1^n0^n$.

This is not a semantic property:

* It takes my program $n$ steps to generate the first 100 strings in $L$.

Note the importance of differentiating between a semantic property and not:

* The halting problem is actually decidable for
[Linear Bouned Automata](https://en.wikipedia.org/wiki/Linear\_bounded\_automaton).

### "Non-Trivial"

A trivial property is a property that either all languages have or no language
has. A non-trivial property is everything else.

### "Undecidable"

A program can either **accept**, **reject**, or **run forever**. If a program
reaches an accept or reject state, we say it **halts**.

There are two types of programs: **recognizers** and **deciders**.

A recognizer is a program that can only tell you with certainty when it has
succeeded to solve a problem (reached the accept state). It cannot always tell
you when it has failed (if it goes into an infinite loop, there is no way to
know if it's in a loop, or if it's just taking very long to solve the problem).

A decider is a program that always reaches either accepts or rejects. That is,
you not only know when the problem was solved, but you also know when it was
*not* solved.

A language is **recognizable** if there exists at least one program that can
recognize it. For example, the following program reconizes $L = \\{"342"\\}$ (the
language made up of only the string "342"):

1. Read input.
2. If input == "342" print "accept".
3. Else return to step 1.

This program *recognizes* $L$, but it does not *decide* $L$: if the input is in
$L$, it accepts, but if it's not, then it will run forever, and you will never
know whether the input was not in $L$ or the program is just taking a long
time.

A language is **decidable** if there exists a program that can decide it. All
decidable languages are also recognizable. Here is a program that decides $L$:

1. Read input.
2. If input == "342" print "accept".
3. Else print "input rejected".

---

## The Halting Problem

Take the following language:

> $HALT\_{ TM } = \\{ \langle M, w \rangle | M$
> is a program and $M$ halts on input $w \\}$

Remember, a program is itself just a string: any program can be written down as
a description, say an `.rb` file, and that file can be used as an input for
another program (or itself!). So $HALT\_{ TM }$ is a language that consists of
all programs $M$ and inputs $w$ such that $M$ halts on $w$.

I won't prove it in this post, but, as it turns out, $HALT\_{ TM }$ is
undecidable. Meaning it is not possible to write a program that decides whether
an algorithm halts.

With this in mind, we can finally prove Rice's theorem:

---

## Rice's Theorem

Recall the theorem:

> Every non-trivial semantic property of a program is undecidable.

Yet another way of stating this is as follows:

> The language $P\_{ TM }$, described below, is undecidable:
> $P\_{ TM } = \\{ \langle M \rangle | M$ is a program and $L(M)$ has
> non-trivial property $P \\}$. Where $L(M)$ means "The language of $M$".

So, for example, it is not possible to write a program $R $ that takes as its
input another program $M$ and decides whether the language of $M$ is regular
(that is, if $M$ can be simplified and represented as a finite automation).

---

## Proof

We can prove Rice's theorem by contradiction. We will show that **if** $P\_{ TM
}$ is decidable **then** so is $HALT\_{ TM }$.  Since we know that $ HALT\_{ TM
}$ is undecidable, then $P\_{ TM }$ must be undecidable too.


Assume that $P$ is some non-trivial semantic property and that it is possible
to write a program $R$ that decides $P\_{ TM }$. Here is how we could solve the
halting problem with that program:

First, we write a program $T$ such that $\langle T \rangle $ is in $P\_{ TM }$.
Because $P$ is non-trivial, such a program must exist.

Take input $\langle M, w \rangle$ and use it to write a program $M\_w$that
takes $x$ as its input and does the following:

**$M\_w $:**

1. Run $M$ on input $w$. If $M$ halts, move on to step 2.  2. Run $T$ on $x$. Accept if $T$ accepts, and reject if $T$ rejects.

Here's the clever part. *We don't actually have to run $M\_w$*:

All we need to know is that, if we *were* to run $M\_w$, there are two possible
outcomes:

* $M$ halts on input $w$, in which case $M\_w$ reaches step 2.
* $M$ never halts and never reaches step 2.

But note that, if $M$ halts on $w$, then step 2 is simply to run $T$, which
means that **when $M$ halts on $w$, $ \langle M\_w \rangle$is in $ P\_{ TM
}$.**

So, if we were to run $R$ with input $\langle M\_w \rangle$, it would be able
to tell us whether it is in $P\_{ TM }$, and that in turn would tell us if $M$
halts on $w$.

But this would mean that we could solve the halting problem, which we know  is
not possible.
