# Build the Popcorn Compiler

## Install pre-requisites and download the source
First, install prerequisites:
```
❯ sudo apt-get install build-essential flex bison git cmake zip texinfo
```
Download the source code:
```
❯ git clone -b main --single-branch https://github.com/ssrg-vt/popcorn-compiler.git
```
## Build the popcorn compiler
```
❯ mkdir build
❯ ls
build  popcorn-compiler
❯ cd popcorn-compiler
❯ ./install_compiler.py --install-all --threads 8 --install-path=../build --with-popcorn-kernel-5_2
```
The `install_compiler.py` will take about *1h* to download and install `LLVM/clang`, `binutils`, `musl-libc` and `libelf` depending on your machine configuration and network speed.

The compiler, including supporting libraries and tools, is now installed under the `build` directory. You can use the the provided Makefile in "popcorn-compiler/util/Makefile.template" to build applications. The makefile template expects that the entire application's source is contained in a single directory.

## Build applications with popcorn compiler

