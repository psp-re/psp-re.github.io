+++
title = "Choose your words wisely"
date = 2021-03-23
description = "Followup to NID Cracking"
[extra]
author = "Paul Sajna"
+++

> “Words have a magical power. They can either bring the greatest happiness or the deepest despair.” -Sigmund Freud

## The problem

What went wrong when I initially tested my Rust nidcracker program? As I 
mentioned in the original post, high entropy and a small hash == garbage output.
The high entropy problem comes from using every permutation of 4 words from 
1000 words (give or take).

## The solution

How can we solve it? Simple, use a shorter, more carefully crafted wordlist.
User Meeeow in 
[The PSP Homebrew Discord Server](https://discord.gg/bePrj9W), 
One of the main contributors behind [UOFW](https://github.com/uofw/uofw), used 
my same exact nidcracker program from the previous post, combined with some
knowledge of what the functions with unknown NIDs did, chose specific words 
related to what the functions were doing, and successfully cracked a few unknown 
NIDs. 

```
found match: 0x7939C851 = sceChkregGetPspModel
found match: 0x6894A027 = sceChkregGetPsFlags
```

## Lessons

The first lesson is, with the right wordlist, these NIDs can be cracked,
and there's absolutely nothing wrong with my program, but with the approach
I took to creating the wordlist for it. In the first blog post, I basically
alleged that NIDs couldn't be cracked without the results being meaningless.
I now know this is not the case. In fact, watch what happens if I cheat a bit
and use known nids, with a wordlist built from words in those known NIDs.

```
found match: ED1410E0 = sceKernelDeleteFpl
found match: 86255ADA = sceKernelDeleteMbx
found match: 6B2371C2 = sceKernelDeleteModule
found match: F8170FBE = sceKernelDeleteMutex
found match: 28B6489C = sceKernelDeleteSema
found match: 9FA03CD3 = sceKernelDeleteThread
found match: 32BF938E = sceKernelDeleteTlspl
found match: 89B3D48C = sceKernelDeleteVpl
found match: 3FC9AE6A = sceKernelDevkitVersion
found match: D636B827 = sceKernelDipswAll
found match: 5282DD5E = sceKernelDipswSet
found match: D774BA45 = sceKernelDisableIntr
found match: 1C46158A = sceKernelDmaExit
found match: 4D6E7305 = sceKernelEnableIntr
found match: 05572A5F = sceKernelExitGame
...
```

Within seconds, we have hundreds of valid nids. (list shortened for blogpost) 

That brings me to the second
lesson: test the happy path before giving up. A coworker of mine says the 
happy path hides bugs, because it's usually the things you don't think about
that cause problems in code. But, if we never test the happy path at all,
how can we ever be happy? In the previous blog post, I tested NIDs that had
been uncracked for years, with 1000 words from NIDs we knew, and concluded
that a solution was impossible. I failed to ever test the happy path, and I
was unhappy with the outcome. It seems obvious now. 

In conclusion:
> Don't worry, be happy. -Bob Marley

PS: I wrote a [hashcat module for NIDs](https://github.com/pspdev/hashcat) after
I was inspired by the successful outcomes of Meeeow.
