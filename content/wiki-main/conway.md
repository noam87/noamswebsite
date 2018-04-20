+++
date = "2018-04-10"
title = "Conway's Game Of Life In 298 Characters Of Julia"
draft = false
wikis = ["computers", "cs"]
tags = ["featured", "julialang", "conway", "automata"]
+++

Yesterday I got distracted playing around with Julia and ended up making this
little Conway's Game of Life. Would fit into a tweet if not for the
pretty-printing / animation loop!.

Works using matrix operations. Parameters by line (uppercase are matrices):

1. `d` = size of board `(dxd)`.
2. S game / map state in 1 and 0 values for on/off.
3. `U` = shift board up if left-multiplied, right if right-multiplied. `D` =
down, left.
4. Clears screen between animation loops.
5. `c` = value of each cell is how many neighbors in `S`.
6. Prettyprint each row in `S`.

You can also change how many generations it will go through (99 currently).

```
d=30
a=1:d;S=[rand(1:5)==1?1:0 for r=a,c=a]
x=(o,t)->[r==o(c,1)||r==t(c,d-1)?1:0 for r=a,c=a];U=x(-,+);D=x(+,-)
for i=1:99;print("\e[2J")
S=map((c,o)->(c==2&&o==1)||c==3?1:0,U*S+S*U+D*S+S*D+U*S*U+U*S*D+D*S*D+D*S*U,S);
for i=a;foldl((p,x)->p*(x==1?"█":".")," ",S[i,a])|>println;end;sleep(0.5);end
```
