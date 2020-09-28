---
title: "Build Clang/LLVM from source on Ubuntu"
classes: wide
excerpt: "What and where to clone the code to build Hipacc."
tags: 
  - clang 
  - toolings 
---
Steps to build clang/llvm and related tools on linux for [Hipacc](https://hipacc-lang.org/).

## Clone source code 

1. clone llvm 

        git clone git@github.com:llvm-mirror/llvm

2. clone compiler-rt, polly, libcxx, libcxxabi 

        cd projects/
        git clone git@github.com:llvm-mirror/compiler-rt
        git clone git@github.com:llvm-mirror/polly
        git clone git@github.com:llvm-mirror/libcxx
        git clone git@github.com:llvm-mirror/libcxxabi
        cd ..

3. clone Clang

        cd tools/
        git clone git@github.com:llvm-mirror/clang

4. clone Clang toolings

        cd clang/
        cd tools/
        git clone git@github.com:llvm-mirror/clang-tools-extra extra
        cd ../../../

   Note that name here must be extra for clang toolings.  

4. git checkout the same release version for the cloned source code  

## Build source code 

1. Create a build directory. Building LLVM in the source directory is not supported. cd to this directory:

        mkdir build
        cd build

2. Use CMake to generate build files, optionally it is possible to set a different install prefix
	
        cmake path/to/llvm/source/root -DCMAKE_INSTALL_PREFIX=/custom/bin/llvm

3. Build and install the binary 

        cmake --build .
        cmake --build . --target install


