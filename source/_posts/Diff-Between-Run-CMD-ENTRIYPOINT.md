---
title: Diffenence between RUN, CMD and ENTRYPOINT in Dockerfile
date: 2022-01-12 22:03:27
tags: Docker
---

# Diffenence between RUN, CMD and ENTRYPOINT in Dockerfile

## 1. RUN

The RUN directive can appear multiple times in Dockerfile, and it only works in build stage.

## 2. CMD and ENTRYPOINT

CMD is similar to ENTRYPOINT, they both work when the container starts. 

But it's quite diffenent.

Let's illustrate its diffenences in this form.

|                                | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”]          |
| :----------------------------- | :------------------------- | :----------------------------- | :--------------------------------------------- |
| **No CMD**                     | *error, not allowed*       | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                            |
| **CMD [“exec_cmd”, “p1_cmd”]** | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd            |
| **CMD [“p1_cmd”, “p2_cmd”]**   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd              |
| **CMD exec_cmd p1_cmd**        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

**In general**

ENTRYPOINT is more recommended than CMD. if you want to run some commands while the container starts, you should write a ENTRYPOINT directive.

### Reference

[Docker Reference](https://docs.docker.com/engine/reference/builder/#cmd)

