+++
draft = false
date = "2017-04-28"
title = "Information Theory Overview"
wikis = ["math", "cs"]
tags = ["informationTheory"]
+++

Roadmap based on [youtube introductory series](https://youtu.be/UrefKMSEuAI?list=PLE125425EC837021F).


| | **Compression** (efficiency, source coding) | **Error Correction** (reliability, channel coding) |
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

## Entropy

How many bits should be needed to send a piece of information?

$$H(X) =  \sum  p\_{i} (log\_{2}( \frac{1}{p\_{1}} )) = E(I(X))$$

## Binary Tree Encoding (Huffman)

<table>
  <thead>
    <th></th>
    <th>$$p\_{i}$$</th>
    <th>Encoded</th>
  </thead>
  <tr>
    <td>"A"</td>
    <td>$\frac{1}{3}$</td>
    <td><code>11</code></td>
  </tr>
  <tr>
    <td>"B"</td>
    <td>$\frac{1}{2}$</td>
    <td><code>0</code></td>
  </tr>
  <tr>
    <td>"C"</td>
    <td>$\frac{1}{12}$</td>
    <td><code>100</code></td>
  </tr>
  <tr>
    <td>"C"</td>
    <td>$\frac{1}{12}$</td>
    <td><code>101</code></td>
  </tr>
</table>

![](/img/bin_tree_encoding.png)
