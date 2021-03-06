# Building ARPACK++ for El Capitan

Armadillo requires an ARPACK installation. I decided to install the ARPACK++ library as well.

## Get ARPACK and arpack++

Download the source from [http://www.caam.rice.edu/software/ARPACK/](http://www.caam.rice.edu/software/ARPACK/). Make sure you get the patch as well.

Untar/gz the ARPACK file in a folder and untar the patch in the same folder. It will overwrite a few files in the distribution, but that's ok. Do the same with arpack++.

## Get SuperLU

A required package is SuperLU, which may be found at [http://crd-legacy.lbl.gov/~xiaoye/SuperLU/](http://crd-legacy.lbl.gov/~xiaoye/SuperLU/). Download the SuperLU version and untar it.

## Install gfortran

clang on OS X does not come with a Fortran compiler, at least not for Xcode version ```Apple LLVM version 8.0.0 (clang-800.0.42.1) ``` (this is what ```gcc -v``` returns).

You can get an Apple installer package from the [gcc wiki](https://gcc.gnu.org/wiki/GFortranBinaries#MacOS).

## Build SuperLU

Use the make.inc file in this repository. Copy it into the top-level folder for SuperLU. Now define

    export CMAKE_SOURCE_DIR=/path/to/SuperLU_5.2.1

Note your version number may be different; use the appropriate folder name.

Now make a build folder in the source tree and use cmake. All the tests should pass. Ignore any warnings you get when making all.

    mkdir build && cd build
    cmake ../ -DCMAKE_INSTALL_INCLUDEDIR=/usr/local/include
    make all
    make test
    sudo make install

## Examine the ARmake.inc file

This file should be modified to point to the source for ARPACK. Save this file in the top level of the ARPACK distribution.

## Make and Install ARPACK

Problems with ```etime``` led to a change to the base ARPACK code. Go to ```UTILS``` and edit ```second.f```. The declaration of ```etime``` should be

    EXTERNAL REAL      ETIME

The original code has two lines but ```gfortran``` would not link unless the lines were replaced with the one above. Save the file.

Type ```make all``` in the ARPACK folder from a Terminal command line. At the end, you will have the library ```libarpack_osx.a``` in the ARPACK folder. Since there is no install target in the makefile, you will have to install it yourself using

    sudo mv libarpack_osx.a /usr/local/lib

There is a note in the ranlib manual that led me to rebuild the library's table of contents after I moved it. So yes, please do

    sudo ranlib /usr/local/lib/libarpack_osx.a

If you don't do this step, the examples may not compile and run. I had to create a symlink in the ARPACK folder for the library

    ln -s /usr/local/lib/libarpack_osx.a libarpack_osx.a

Make a symlink to libarpack.a.

    sudo ln -s /usr/local/lib/libarpack_osx.a /usr/local/lib/libarpack.a

## Install arpack++

Fortunately, arpack++ is a header-only library. This simplifies installation since you can just copy the headers.

    sudo mkdir /usr/local/include/arpack++
    cd include
    sudo cp *.h /usr/local/include/arpack++

## Follow Armadillo Installation Instructions

See [Conrad's Linux/Mac installation instructions](https://github.com/conradsnicta/armadillo-code) and enjoy.

Note: I was not able to build the wrapper, so I just

    sudo cp -R armadillo /usr/local/include
    sudo cp -R armadillo_bits /usr/local/include

from the ```include``` folder. There were several gfortran symbols that could not be found. I was not able to track down why clang didn't find the gfortran library. Attempting to mention it on the g++ link line didn't yield any positive results.

If you find a solution to this, please update this document and send me a pull request.
