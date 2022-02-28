# Build the Popcorn Compiler

Popcorn compiler was tested to build on Ubuntu 18.04. It relies on several dependencies, so it is easier to build using a Dockerfile.

## Build popcorn compiler using docker (recommended)
First, you need to install [docker](https://docs.docker.com/engine/install/). You can find the method to install docker on ubuntu [here](https://docs.docker.com/engine/install/ubuntu/).

Next, download the Dockerfile for popcorn-compiler. You can find the Dockerfile [here](https://raw.githubusercontent.com/ssrg-vt/popcorn-compiler/main/docker/Dockerfile), or download the entire popcorn-compiler repository as follows:
```
❯ git clone -b main --single-branch https://github.com/ssrg-vt/popcorn-compiler.git
❯ cd popcorn-compiler/docker
❯ docker build -t popcorn-compiler:main .
```
Last, you can build a test application using the docker image built from the 2nd step. Consider the application to be built is in `./app`, build the application with:
```
❯ ls ./app
Makefile test.c
❯ docker run --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/app:/code/app popcorn-compiler:main \
    make -C /code/app
make: Entering directory '/code/app'
 [MKDIR] build_aarch64/
 [CC] test.c
 [LD] build_aarch64/test_aarch64 (vanilla)
 [MKDIR] build_x86-64/
 [LD] build_x86-64/test_x86-64 (vanilla)
 [ALIGN] build_aarch64/aligned_linker_script_arm.x
 [LD] test_aarch64 (aligned)
 [ALIGN] build_x86-64/aligned_linker_script_x86.x
 [LD] test_x86-64 (aligned)
 [POST_PROCESS] test_aarch64 test_x86-64
make: Leaving directory '/code/app'
❯ ls ./app
build_aarch64  build_x86-64  Makefile  test_aarch64  test_aarch64.o  test.c  test_x86-64  test_x86_64.o
```

Next, follow this [link](run_applications#run-the-pingpong-application) to run the popcorn applications.

## Build popcorn compiler manually
### Install pre-requisites and download the source code
First, install prerequisites:
```
❯ sudo apt-get install build-essential flex bison git cmake zip texinfo
```
Download the source code:
```
❯ git clone -b main --single-branch https://github.com/ssrg-vt/popcorn-compiler.git
```
### Build the popcorn compiler
```
❯ mkdir build
❯ ls
build  popcorn-compiler
❯ cd popcorn-compiler
❯ ./install_compiler.py --install-all --threads 8 --install-path=../build --with-popcorn-kernel-5_2
```
The `install_compiler.py` will take about 1h to download and install `LLVM/clang`, `binutils`, `musl-libc` and `libelf` depending on your network speed.

The compiler, including supporting libraries and tools, is now installed at `build`. You can use the the provided Makefile in "popcorn-compiler/util/Makefile.pyalign.template" to build applications. The makefile template expects that the entire application's source is contained in a single directory.

