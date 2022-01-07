
---
title: "Logging impact application performance"
date: 2022-01-09
publishdate: 2022-01-09
authors: Liran Mendelovich
img: docs/liran.jpg
draft: true
tags: ["performance", "scale"]
layout: section
---

# Logging impact application performance

## Highlights

* Invalid log writing
* Real high scale execution recording showing the effect
* -> Why valid log writing is very important. 

## Story

Consider the following code:

```
parseMessage(List<Message> messages) {
    log.debug("messages: " + messages);
}
```

or on a different version:

```
parseMessage(List<Message> messages) {
    log.debug("messages: {}", messages.toString());
}
```

What is the issue with this code line of log writing ?

This resulting with toString() method invocation on each call, even when log verbosity resulting with nothing written in log. This method invocation is redundant, and when written in such processing methods which are executed per message, can result in performance degradation in high scale executions.

## Real high scale execution recording showing the effect

FILL IN

## Solution

Solution is valid code writing which will cause toString() method invocation only when needed, for example when log is in debug verbosity: 

```
parseMessage(List<Message> messages) {
    log.debug("messages: {}", messages);
}
```

