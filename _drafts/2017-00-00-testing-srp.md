---
layout: post
title: Testing SRP
---

## Testing with other implementations

In order to verify my implementation, I wanted to test it against other implementations. I started with the Python package [srp](https://github.com/cocagne/pysrp). However after some testing I found that it didn't implement the current version of SRP. So I replaced that package with the Python package [srptools](https://github.com/idlesign/srptools). After some initial testing I found that this package implemented the same version.

I wanted to write unit tests in Swift, which then called the Python implementation. So I wrote a Python script that could be communicated with using stdin, stdout and stderr. In order to ease writing the tests, I created a 