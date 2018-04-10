+++
date = "2018-04-10"
title = "Conway's Game Of Life In 304 Characters Of Julia"
draft = false
wikis = ["computers", "cs"]
tags = ["featured", "julialang", "conway", "automata"]
+++

Yesterday I got distracted playing around with Julia and ended up making this
little Conway's Game of Life. Would fit into a tweet if not for the pretty-printing / animation loop!.

Works using matrix operations:

* `U` = shift board up if left-multiplied, right if right-multiplied.
* `D` = down, left.

Starts with randomly generated board with parameters:

* `d` = size of board.
* `p` = initial live cell density (the higher, the less dense).

You can also change how many generations it will go through (100 currently).

```
d=50;p=50;s=[rand(1:p)==1?1:0 for r=1:d,c=1:d]
x=(l,o,t)->[r==o(c,1)||r==t(c,(l-1))?1:0 for r=1:l,c=1:l];U=x(d,-,+);D=x(d,+,-)
for i=1:100;print("\e[2J")
s=map(c->c==2||c==3?1:0,U*s+s*U+D*s+s*D+U*s*U+U*s*D+D*s*D+D*s*U);for i=1:d
foldl((p,x)->p*(x==1?"█":" ")," ",s[i, 1:d])|>println;end;sleep(0.5);end
D
```

