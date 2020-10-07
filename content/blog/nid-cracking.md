+++
title = "NID Cracking"
date = 2020-10-17
description = "Fun with hash collisions"
[extra]
author = "Paul Sajna"
+++

## What is a NID?

A NID, which I believe stands for Name Identifier, is an identifier used in Sony's PRX
executable format for function imports and exports. They are located in the .rodata.SceNid
section of every PRX, to identify which operating system or other external functions are
used in the program. A NID consists of the first 4 little-endian bytes of the SHA-1 hash
of the function name, and in later versions of the PSP OS, a salt.  Legend has it, 
they can be bruteforced to discover the names Sony used for their functions, to aid in
reverse engineering. Experienced cryptographers would have probably realized the problem
before writing a program to check the results. 

## The problem

So it turns out, there are quite a few hash collisions for the first 4 hexadecimal bytes 
of a SHA-1 hash. Here are a few of my favourites, which you can check by running 
`echo -n <name> | sha1sum`
on any linux pc (assuming checksum packages are installed). Note that you have to swap 
endianness (read the bytes backwards). 

```
7CF05E81 = sceSysregAllocateNotifyRapidHead
4C0BED71 = sceSysregAbortFatfmtSteepIn
7CF05E81 = sceSysregActivateDataMbogoPIO
```
I purposely chose two here that came up with the same NID for the function name, to 
demonstrate the problem of hash collisions. If you're following along at home, run the 
following at a linux prompt:

```
echo -n sceSysregAllocateNotifyRapidHead | sha1sum
echo -n sceSysregActivateDataMbogoPIO | sha1sum
```

The results are

```
815ef07c4b2f44c74cab9af3772c49e49dc41544  -
815ef07cbd77ac3b13f1813c063aff7794960741  -
``` 
It is plain to see  that they start with the same 4 bytes (8 characters), and if you
read the bytes (digit pairs) backwards, you will see the NID. So that's no good, 
we have multiple possible answers to the same problem, and both of them are nonsensical. 
I'm unsure how functionnames were ever recovered from NIDs, after working through this.
I know some of the function names were leaked in [games with debug symbols](http://psp.re/blog/list/),
but that doesn't account for all the kernel functions that were never used in games
that have been recovered. Maybe the salts narrow the problem down a bit, I'm not sure.
I tested without salts, because I was told earlier versions of the kernel didn't use them,
with NIDs, and it was easier for me not to.

## Methods

So, if you want to try your hand at NID cracking, I'll explain what I did in this section.
I wrote a parallelized [Rust](https://rust-lang.org) program that runs on a modern
desktop CPU. I was planning to later use GPUs for more parallelism/speed, but the 
ridiculous results I got have me convinced it's not worth persuing further. 

If you're unfamiliar with Rust, you can download it at [https://rustup.rs](https://rustup.rs), and learn how to use it at [https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/)

The (slightly messy) source of my program is below:

```rust
use openssl::sha::sha1;
use rayon::prelude::*;
use rayon::iter::ParallelBridge;
use std::fs::read_to_string;
use itertools::Itertools;

fn main() {
    //let module_names_string = read_to_string("module-names.txt").unwrap();
    //let module_names_split = module_names_string.split('\n');
    //let module_names: Vec<&str> = module_names_split.collect();

    let module_name = "sceSysreg";

    let wordlist_string = read_to_string("wordlist.txt").unwrap();
    let wordlist_split = wordlist_string.split('\n');
    let wordlist: Vec<&str> = wordlist_split.collect();

    let unknown_nids_string = read_to_string("unknown_nids.txt").unwrap();
    let unk_nids_split = unknown_nids_string.split('\n');
    let unk_nids: Vec<&str> = unk_nids_split.collect();

    wordlist.par_iter().for_each(|word| {
        let test_string = module_name.to_string() + word;
        let hash = sha1(test_string.as_str().as_bytes());
        let test_nid = format!(
        "{:02X}{:02X}{:02X}{:02X}",
        hash[3], hash[2], hash[1], hash[0],
        );
        if unk_nids.contains(&test_nid.as_str()) {
            println!("found match: {} = {}", test_nid, test_string);
        }
    });

    let wordlist_2perms = wordlist
        .iter()
        .permutations(2)
        .into_iter()
        .par_bridge()
        .into_par_iter();

    wordlist_2perms.for_each(|perm| {
        let test_string = module_name.to_string() + perm[0] + perm[1];
        let hash = sha1(test_string.as_str().as_bytes());
        let test_nid = format!(
        "{:02X}{:02X}{:02X}{:02X}",
        hash[3], hash[2], hash[1], hash[0],
        );
        if unk_nids.contains(&test_nid.as_str()) {
            println!("found match: {} = {}", test_nid, test_string);
        }
    });

    let wordlist_3perms = wordlist
        .iter()
        .permutations(3)
        .into_iter()
        .par_bridge()
        .into_par_iter();

    wordlist_3perms.for_each(|perm| {
        let test_string = module_name.to_string() + perm[0] + perm[1] + perm[2];
        let hash = sha1(test_string.as_str().as_bytes());
        let test_nid = format!(
        "{:02X}{:02X}{:02X}{:02X}",
        hash[3], hash[2], hash[1], hash[0],
        );
        if unk_nids.contains(&test_nid.as_str()) {
            println!("found match: {} = {}", test_nid, test_string);
        }
    });

    let wordlist_4perms = wordlist
        .iter()
        .permutations(4)
        .into_iter()
        .par_bridge()
        .into_par_iter();

    wordlist_4perms.for_each(|perm| {
        let test_string = module_name.to_string() + perm[0] + perm[1] + perm[2] + perm[3];
        let hash = sha1(test_string.as_str().as_bytes());
        let test_nid = format!(
        "{:02X}{:02X}{:02X}{:02X}",
        hash[3], hash[2], hash[1], hash[0],
        );
        if unk_nids.contains(&test_nid.as_str()) {
            println!("found match: {} = {}", test_nid, test_string);
        }
    });
}
```

I hard-coded the module name to look for functions of the sceSysreg module for this first
test, and was planning to cover other modules as well. The program then uses rayon's parallel iterator and itertools to calculate permutations of a wordlist, and calculates hashes of the permuations with openssl. I used a python program to build a wordlist from known nids (basically splitting camelcase into words), and got some unknown nids from the emulator 
[JPCSP](https://github.com/jpcsp/jpcsp/blob/d4c891ec1e9ba820a70a9f17ba0af3295b593c6b/src/jpcsp/HLE/modules/sceSysreg.java). My wordlist is at [https://psp.re/wordlist.txt](https://psp.re/wordlist.txt) and the nids I chose to crack are at [https://psp.re/unknown_nids.txt](https://psp.re/unknown_nids.txt)

Rust projects also have a Cargo.toml file with basic metadata, which I will supply below

```toml
[package]
name = "nidcracker"
version = "0.1.0"
authors = ["Paul Sajna <sajattack@gmail.com>"]
edition = "2018"

[dependencies]
openssl = "0.10"
rayon = "1.4.1"
itertools = "0.8.2"
```

Build the project like any other rust project, with `cargo build --release`, and the binary
will be output as target/release/nidcracker.

So I ran all this for a few hours, and you can see my results at [https://gist.github.com/sajattack/dd4c6df4aec0de8b641216515bdc49ed](https://gist.github.com/sajattack/dd4c6df4aec0de8b641216515bdc49ed)

## Conclusion

I'm not really sure where to go from here. The results are nonsensical so I will have to
find another way to uncover kernel function names. If you have any suggestions feel free
to contact me on [Twitter](https://twitter.com/sajattack).
