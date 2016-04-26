# Example how to build a C++ project using Clang (in Linux, Ubuntu), CMake, CLion and Conan

This project is able to build a C++ app that depends on Poco and Boost libraries.
It will use Conan to install both libraries and their transitive dependencies.

Typical linux development uses gcc compiler, and many of the conan packages already have pre-built binaries
for gcc. This guide shows how to built them with Clang.

## Setup

This example has been developed and tested in Ubuntu 14.04

Install Clang compiler:

```bash
$ sudo apt-get install clang-3.6
```

Ensure that you have a CMake > 3.0 version in your path, install it otherwise.

```bash
$ cmake --version
```

Install conan (the simplest way is using python/pip):

```bash
$ pip install conan
```

Make a clone of this project:

```bash
$ git clone https://github.com/memsharded/four-c-example.git
$ cd four-c-example
$ mkdir build && cd build
```

## Installing dependencies with Conan

Typically, the package maintainers in conan creates package binaries for the most used and mainstream platforms, and some package binaries as those for clang in linux might be missing for some packages. Then, the --build option has to be specify to build the package binaries from sources locally.

First, we have to make sure that we are using the Clang compiler, so packages that are being built are built with the right compiler and do not fallback to use the gcc compiler. Some conan packages might be able to check that the correct compiler is being used, and warn otherwise (e.g. if using cmake), but other packages would simply fail to build, like boost, that requires "clang++" to be on the path (there might be other ways to do it with bjam files, but are a bit more complicated and current conan package does not support it):

```bash
//for packages using cmake and unix makefiles
$ export CC=clang-3.6
$ export CXX=clang++-3.6
// For boost to work
$ sudo ln -s /usr/bin/clang++-3.6 /usr/bin/clang++
```

Now we are ready to install all the dependencies. It is retrieving and building Boost, Poco and its
transitive dependencies, like OpenSSL and Zlib, so you might want to go and grab a coffe...

```bash
$ conan install .. -s compiler=clang -s compiler.version=3.6 --build
```

## Building with CMake

Given that we have already defined the environment variables, it is easy to build the project with cmake:

```bash
$ cmake .. -DCMAKE_BUILD_TYPE=Release
$ cmake --build .
$ bin/timer
Callback called after 249 milliseconds.
Callback called after 749 milliseconds.
Callback called after 1250 milliseconds.
Callback called after 1750 milliseconds.
Callback called after 2250 milliseconds.
Callback called after 2750 milliseconds.
true
false
```

## Developing with CLion

There are two ways to launch CLion:
1. If you run the clion.sh launch script within your terminal, then it will get your terminal exported
environment variables, and it will be ready to run

2. If you run clion from a desktop launcher or any other mean, it will not use your environment variables
and then it will typically detect and use the system gcc, which will end in conan raising an error, as the
configuration and dependencies have been set to use clang. To define the correct compiler you can do:
    1. Go to Menu->File->Settings->Build, Execution, Deployment->CMake
    2. Add to "CMake options": -D CMAKE_C_COMPILER=clang-3.6 -D CMAKE_CXX_COMPILER=clang++-3.6
    3. Press OK
    4. Go to Menu->Invalidate Cache/Restart, so CLion is able to process it properly


When you are done, you can already use Run menu to build and launch the timer example.


NOTES:

- You can go to the console any time to "conan install" the dependencies you need, or when you change settings
- I am working on a wrapper to launch conan from cmake, which might further ease this process and provide an
improved workflow with CLion
