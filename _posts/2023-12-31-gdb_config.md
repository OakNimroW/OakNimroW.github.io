---
title: GDB Config to Debug
date: 2023-12-31 00:00:00 +/-TTTT
categories: [Config, buffer_overflow]
tags: [buffer_overflow]     # TAG names should always be lowercase
---

# GDB Config

## Syntax

The default syntax is *AT&T*.
`set disassembly-flavor intel` change the syntax to *intel*

```bash
(gdb) set disassembly-flavor intel
```

## Hook-Stop

`hook-stop` is used to execute a list of commands after an breack point.
`info registers` shows the value of registers, `i r` is valid to.
`x/32wx $esp` shows 32 words at the stack.
`x/4i $eip` shows the following 4 assembly instructions.
`end` ends the list of commands

```bash
(gdb) define hook-stop
> i r
> x/32wx $esp
> x/4i $eip
> end
```

