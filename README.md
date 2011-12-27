
node-lapack
===========

A node.js wrapper for the high-performance LAPACK linear algebra library.

Prerequisites
=============

This library require LAPACK to be built and installed as a shared library.
In time the entire build process may be unified into this project, but that's
some time away.

In the meantime I've placed some basic LAPACK build instructions at the end of this document.

Installation
============

    npm install lapack

Usage
=====

    var lapack = require('lapack');

    /*
    LAPACK functions
    */

    var result = lapack.sgeqrf([
        [1, 2, 3],
        [3, 4, 5],
        [5, 6, 7]
    ]);

    console.log(result.R);
    console.log(result.tau);

    result = sgesvd('A', 'A', [
        [1, 2, 3],
        [3, 4, 5],
        [5, 6, 7]
    ]);

    console.log(result.U);
    console.log(result.S);
    console.log(result.VT);

    result = lapack.sgetrf([
        [1, 2, 3],
        [3, 4, 5],
        [5, 6, 7]
    ]);

    console.log(result.LU);
    console.log(result.IPIV);

    /*
    conveniently wrapped processes
    */

    // perform a complete LU factorization
    var lu = lapack.lu([
        [1, 2, 3],
        [3, 4, 5],
        [5, 6, 7]
    ]);

    console.log(lu.L);
    console.log(lu.U);
    console.log(lu.P);


    // perform a complete QR factorization
    var qr = lapack.qr([
        [1, 2, 3],
        [3, 4, 5],
        [5, 6, 7]
    ]);

    console.log(qr.Q);
    console.log(qr.R);

    // solve a system of linear equations

    var a = [
    	[2, 4],
    	[2, 8]];

    var b = [[2], 
    	[4]];

    var result = lapack.sgesv(a, b);
    console.log(result.X);
    console.log(result.P);

LAPACK Building
===============

These instructions are raw and a work in progress, but should work fine for linux and MacOS. The net result will be LAPACK and BLAS shared libraries.

If you haven't done so already download the LAPACK source http://www.netlib.org/lapack/

After unpacking the source create a make.inc based on the make.inc.example provided. For the purposes of this document I'll assume you have the "gfortran" fortran compiler installed. 

Then make the following changes:

make.inc
--------

CHANGE:

    FORTRAN  = gfortran
    OPTS     = -O2
    DRVOPTS  = $(OPTS)
    NOOPT    = -O0
    LOADER   = gfortran
    LOADOPTS =

TO:

    UNAME := $(shell uname)

    FORTRAN  = gfortran
    OPTS     = -O2 -fPIC
    DRVOPTS  = $(OPTS)
    NOOPT    = -O0 -fPIC
    LOADER   = gfortran
    LOADOPTS =

    ifeq ($(UNAME), Darwin)
    LIBEXT=dylib
    else
    LIBEXT=so
    endif

BLAS/SRC/Makefile
-----------------

CHANGE:

    all: $(BLASLIB)

TO:

    all: $(BLASLIB) libblas.$(LIBEXT)

    libblas.$(LIBEXT):
        $(FORTRAN) -shared -o $@ *.o
        cp $@ ../../$@

CHANGE:

    clean:
        rm -f *.o

TO:

    clean:
        rm -f *.o
        rm -f *.a
        rm -f *.$(LIBEXT)

SRC/Makefile
------------

CHANGE:

    all: ../$(LAPACKLIB)

TO:

    all: ../$(LAPACKLIB) liblapack.$(LIBEXT)

    liblapack.$(LIBEXT):
        gfortran -shared -o $@ $(ALLOBJ) -lblas -L..
        cp $@ ../$@

CHANGE:

    clean:
        rm -f *.o

TO:

    clean:
        rm -f *.o
        rm -f *.a
        rm -f *.$(LIBEXT)

Compiling
---------

    make blaslib
    make lapacklib

Installing
----------

    # for linux
    cp liblapack.so libblas.so /usr/lib
    # for macos
    cp liblapack.dylib libblas.dylib /usr/lib