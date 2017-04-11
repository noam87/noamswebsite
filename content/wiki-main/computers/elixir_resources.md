+++
title = "Elixir/Erlang Resources"
draft = false
date = "2017-01-04T16:11:51-05:00"
wikis = ["computers"]
tags = ["elixir", "erlang", "phoenix", "recommendations", "gc"]
+++

## Learning

| Title                                    | Notes  |
|------------------------------------------|--------|
| [Elixir In Action]( http://a.co/hjRDstC) (book) | My go-to recommendation for getting started with Elixir. |
| [Writing A Blog Engine in Phoenix](https://medium.com/@diamondgfx/introduction-fe138ac6079d) | Great first tutorial for Phoenix framework |
| [Erlang In Anger](https://www.erlang-in-anger.com/) | Great guide for when stuff goes bad. |

## Memory / GC

For articles on GC / compilers in general see [compilers](/wiki-main/computers/compilers) section.

| Title | Notes  |
|-------|--------|
| [Erlang Scheduler Details and Why It Matters](https://hamidreza-s.github.io/erlang/scheduling/real-time/preemptive/migration/2016/02/09/erlang-scheduler-details.html) | |
| [Erlang Garbage Collection Details and Why It Matters](https://hamidreza-s.github.io/erlang%20garbage%20collection%20memory%20layout%20soft%20realtime/2015/08/24/erlang-garbage-collection-details-and-why-it-matters.html) | Simple introduction to Erlang GC concepts. Start here. |
| [Erlang 19.0 Garbage Collector](https://www.erlang-solutions.com/blog/erlang-19-0-garbage-collector.html) | |
| [Erlang Efficiency Guide](http://erlang.org/doc/efficiency\_guide/introduction.html) | |
| [Erlang Binary Garbage Collection: A love/hate relationship](http://blog.bugsense.com/post/74179424069/erlang-binary-garbage-collection-a-lovehate) | Some findings on the shared heap GC. |

## Concurrency

| Title | Notes  |
|-------|--------|
|[How Erlang does scheduling](http://jlouisramblings.blogspot.ca/2013/01/how-erlang-does-scheduling.html?m=1) | |
|[Characterizing the Scalability of Erlang VM on Many-core Processors](http://kth.diva-portal.org/smash/record.jsf?searchId=2&pid=diva2%3A392243&dswid=-8162) (pdf) | |

## Performance

| Title                                    | Notes  |
|------------------------------------------|--------|
| [Elixir RAM And The Template Of Doom](http://www.evanmiller.org/elixir-ram-and-the-template-of-doom.html) | |
| [Elixir and IO Lists, Part 1: Building Output Efficiently](https://www.bignerdranch.com/blog/elixir-and-io-lists-part-1-building-output-efficiently/) | |

## Performance Tips

Don't have processes the many other processes depend on. For example, a
single-process cache that many others call to. This will be a bottleneck
because you'll always have to wait for that process's turn from the scheduler.
