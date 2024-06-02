+++
title = "PSP RE quick start guide"
template = "index.html"
+++

## Introduction

This short guide will present you all the necessary ressources you need to start reverse
engineering on PSP. This guide works for both reverse engineering kernel modules and
userland (most probably game) binaries.

It is supposed you already know how to reverse engineer: if you don't, check a ghidra
tutorial and preferably also a MIPS assembly course.

Note there exists alternatives to Ghidra, for example using IDA Pro. Ghidra is used here because:
- its ecosystem is more mature, especially for PSP ;
- it's opensource and easy to install ;
- it has convenient features such as decompilation, etc.

## Step 1: getting a binary to RE

In order to get a binary to decrypt, you first need to pick:
- the game binary, located in `psp_game/sysdir/` of the game ISO, either `boot.bin` (unencrypted) or `eboot.bin` (encrypted) ;
- or an updater .PBP, if you want to reverse engineer the kernel.

In both cases, if you encounter a .PBP file or an encrypted file, you can use the
[pspdecrypt](https://github.com/John-K/pspdecrypt) tool to decrypt it.

## Step 2: setting up Ghidra

Install Ghidra and install [ghidra-allegrex](https://github.com/kotcrab/ghidra-allegrex) using
the included README. Note there are some limitations in the plugin: VFPU support is limited,
64-bit return values or arguments (using two registers) are not properly handled.

After Ghidra is setup, you can start it and install the
[psp-ghidra-scripts](https://github.com/pspdev/psp-ghidra-scripts) using the included README
in order to fix function imports and exports.

Then, you can download the [pspsdk.gdt](/pspsdk.gdt) type archive for games, or
[uofw.gdt](/uofw.gdt) for kernel modules (or both, but that might cause incompatibilities).

These files were built using include files from the PSPSDK or uOFW using the `File` -> `Parse C Source...` menu.
If you want to regenerate (or update) them:
- for the PSPSDK, you need to insert in `Source files to parse`, from the `src` folder,
`base/psptypes.h`, `debug/*.h`, `user/pspkerneltypes.h`, `user/*.h`, then the rest of the `.h` files
- for uOFW, you need to insert `include/common/*.h`, then `include/*.h`

## Step 3: analyzing the binary

Now that Ghidra is up and running, you should:
1. Import the (decrypted) binary file in Ghidra
2. Run "Analyze" when prompted for it
3. Run the `SonyPSPResolveNIDs.py` script
4. Open the .gdt file with `Menu -> Open File Archive...` in the `Data Type Manager`
5. Right click on the imported data type archive and use `Apply Function Data Types`

Now all the imports should use the correct signatures, and you're good to go!

