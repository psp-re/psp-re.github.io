+++
title = "Why Reverse Engineer The PSP?"
date = 2020-07-13
description = "A summary of the reasons this website exists"
[extra]
author = "Paul Sajna"
+++

## Background

The Sony Playstation Portable handheld game console is iconic partially because of it's
prolific homebrew scene. It was the first game console that I personally owned as a kid,
and perhaps in some way it inspired me to become a software engineer. At the very least, 
it inspired me to hack, and hack it I did. Shortly after I got the PSP, I brought it to
a random electronics consultant in my hometown to perform the Pandora's Battery hack
which required a soldering iron I did not have at the time. I know many others who
had similar experiences. 

## Rust and Puzzle Bobble

Lately I've been working on a project called 
[rust-psp](https://github.com/overdrivenpotato/rust-psp). It's exactly what you think 
it is, a rust-lang toolchain for making PSP Homebrew. We have achieved parity
with the unofficial C SDK for user-mode applications. 
What does an enterprising rust hacker do when he has met C? Go further. It all starts
with Puzzle Bobble. While reading the C SDK to translate it to Rust, I came across an
interesting comment in the source code:
```
/* Note: Some of the structures, types, and definitions in this file were
   extrapolated from symbolic debugging information found in the Japanese
   version of Puzzle Bobble. */
```
Debug information in Puzzle Bobble you say? Fascinating. After seeing this comment a few
times in various files, I couldn't help but crack it open in radare2/Cutter. I had 
previously attempted reverse engineering 
[C64 Pacman](https://github.com/sajattack/c64-pacman-disassembly), so I knew my way
around the tool, and I was waiting on a PR to be merged before I could implement
rust-std for PSP. So I cracked it open and found entire libraries that were not yet
in the C SDK. Eventually radare2/Cutter became limiting and I switched to Ghidra.

## Today Puzzle Bobble, Tomorrow The World!

I reverse engineered sceGuDebugPrint from Puzzle Bobble, and there are many things in 
Puzzle Bobble I still have to work on, but I couldn't help but wonder, "How many other
games like this are there?" The answer I've found so far is 12. If you find more,
please let me know. 

## K great, but why did you make a website about it?

I made this website to share findings with the community and have a central place
for this information on the web. You are welcome to contribute on github.  
