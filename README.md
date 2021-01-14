# osqp-matlab-cmake-buildsystem

## Overview

This repo contains a drop-in CMake buildsystem for the osqp-matlab bindings mantained at https://github.com/oxfordcontrol/osqp-matlab .

**If you are just interested in using OSQP in MATLAB, please follow the official instructions at https://osqp.org/docs/get_started/matlab.html.**

This repo is only useful if for any need to compile the MATLAB extension against a OSQP library that is already in your system, for example
if you are also using OSQP in other software that is loaded by MATLAB, and you want to avoid ABI incompatibilities.

**At the moment, this CMake build system does not support the osqp codegen functionalities. If you need that, please use the official binaries.**

## Usage

How to use this repo

1. Install osqp in your system, and make sure that it can be find by CMake.

2. Clone the repository

~~~
git clone git://github.com/dic-iit/osqp-matlab-cmake-buildsystem
~~~

3. Build it

~~~
cd osqp-matlab-cmake-buildsystem
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=<install-prefix> ..
cmake --build . --config Release
cmake --install . --config Release
~~~

4. Use it

Add the `<install-prefix>/mex` directory to the MATLAB path.

## License

Materials in this repository are distributed under the following license:

> All software is licensed under the Apache-2 License. See [LICENSE](./LICENSE) file for details.

## FAQ

### How the version is chosen?

The version of this `CMake` project is chosen in accordance of the original project, plus a fourth version that describes if several CMake buildsystem
version were release, for example the version `x.y.z.t` is the one corresponding to the `x.y.z` version of osqp-matlab, while `t` is the number that can be
increase is changes are done to the CMake buildsystem.
