+++
title = "Elixir/Erlang Performance Tips"
draft = false
date = "2017-01-14"
wikis = ["computers"]
tags = ["elixir", "erlang"]
+++

Don't have processes the many other processes depend on. For example, a
single-process cache that many others call to. This will be a bottleneck
because you'll always have to wait for that process's turn from the scheduler.

---
