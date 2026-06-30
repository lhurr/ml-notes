---
title: KV Cache
---

## Overview

The KV cache stores past keys and values so [[Prefill and Decode|decode]] does not need to recompute attention over the whole sequence every step.

## Why
- In attention, token `t` attends to all previous tokens via their **keys** and **values**.
- Without a cache, each decode step would recompute K and V for *every* past token.
- The K and V for past tokens never change, so cache them once and reuse → each step is O(n).

