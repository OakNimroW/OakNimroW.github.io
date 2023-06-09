---
title: Boot Linux Manually
date: 2023-05-16 00:00:00 +/-TTTT
categories: [Something]
tags: [Something]     # TAG names should always be lowercase
---


# Boot Linux Manually

```text
grub> ls
grub> ls (hd0,gpt4)/@/
grup> set prefix=(hd0,gpt4)/@/boot/grub
grub> set boot=(hd0,gpt4)
grub> insmod normal
grub> normal
```

