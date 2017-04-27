+++
draft = false
date = "2017-04-17"
title = "Information Theory Overview"
wikis = ["math", "cs"]
tags = ["informationTheory"]
+++

Roadmap based on [youtube introductory series](https://youtu.be/UrefKMSEuAI?list=PLE125425EC837021F).


| blank | **Compression** (efficiency, source coding) | **Error Correction** (reliability, channel coding) |
|-------|---------------------------------------------|----------------------------------------------------|
| Information Theory (Math) | **Losless:** source coding theorem,  Kraft-McMillan inequality | |
|                           | **Lossy:** rate distortion theorem | |
| Coding Methods (algorithms) | **Symbol Code:** Huffman codes | Hamming codes, BCH codes, Turbocodes, Gallager codes |
|                             | **Stream Codes:** arithmetic coding | |

## Information

How many bits are needed to encode information:

$$log\_{2}(\frac{1}{p})$$

Where $p$ is the probability of the event. For example, the number of bits
needed for encoding the value of the result of a coin flip ($p=\frac{1}{2}$)
is 1.

## Binary Tree Encoding (Huffman)

![](/img/bin_tree_encoding.png)
