---
title: 'mlpack 4: a fast, header-only C++ machine learning library'
tags:
  - machine learning
  - deep learning
  - c++
  - header-only
  - efficient

authors:
 - name: Ryan R. Curtin
 - name: Marcus Edel
 - name: Shubham Agrawal
 - name: Suryoday Basak
 - name: James J. Balamuta
 - name: Ryan Birmingham
 - name: Kartik Dutt
 - name: Dirk Eddelbuettel
 - name: Rishabh Garg
 - name: Shikhar Jaiswal
 - name: Aakash Kaushik
 - name: Anjishnu Mukherjee
 - name: Nanubala Gnana Sai
 - name: Kim SangYeon
 - name: Nippun Sharma
 - name: Omar Shrit
 - name: Yashwant Singh
 - name: Roshan Swain
 - name: Conrad Sanderson

affiliations:
date: 4 November 2022
bibliography: paper.bib

---

# Summary

For over 15 years, the mlpack machine learning library has served as a "swiss
army knife" for C++-based machine learning [@curtin2013mlpack,
@curtin2018mlpack].  Its efficient implementations of common and cutting-edge
machine learning algorithms have been used in a wide variety of scientific and
industrial applications.  This paper serves to introduce mlpack 4, the newest
major version of the library, which has been significantly refactored and
redesigned to allow an easy prototyping-to-deployment pipeline, including
bindings to other languages (Python, Julia, R, Go, and the command line) that
allow prototyping to be seamlessly performed in environments other than C++.

# Statement of Need

Over the past few decades, the use of machine learning has become ubiquitous in
almost every scientific discipline and countless commercial applications
[@jordan2015machine] [@carleo2019machine].  There is one important commonality
to virtually every one of these applications: machine learning---which is built
on top of linear algebra operations---is computationally intensive.  Modern
large language models such as BERT [@devlin2018bert] have hundreds of millions
of training parameters, and correspondingly takes a massive amount of
computation to train.  Even non-deep learning approaches such as random forests
may take many hours to train due to their extensive computational needs
[@gieseke2018training].  Clearly, this highlights the need for efficient
implementations, and this was the clear motivator for the original development
of mlpack in C++ [@curtin2013mlpack].

However, fast implementations are only one piece of the puzzle.  A machine
learning application cannot be a success if the process of deploying the model
is too difficult or unwieldy [@paleyes2020challenges, @lavin2022technology];
and, often, deployment environments may have computational or engineering
constraints that make a full-stack Python solution infeasible [@fischer2020ai].
It is therefore quite important that lightweight, easy-to-deploy machine
learning solutions be available---and this has motivated our refactoring and
redesign of mlpack 4: we pair efficient implementations with easy and
lightweight deployment, making mlpack suitable for a wide range of deployment
environments.  A more complete set of motivations can be found in the mlpack
vision document [@mlpack2021vision].

# Functionality

mlpack contains a wide variety of machine learning algorithms, some of which are
new to mlpack 4.  These include "traditional" algorithms such as linear
regression, logistic regression, and random forests, as well as algorithms not
implemented in other libraries, such as algorithms for furthest-neighbor search
[@curtin2016fast], accelerated k-means variants [@curtin2017dual], kernel
density estimation (new) [@lee2008fast], and fast max-kernel search
[@curtin2014dual].  mlpack also contains a deep neural network module, including
implementations of numerous layer types, activation functions, and reinforcement
learning applications.  All of this functionality is detailed in the [mlpack
documentation](https://www.mlpack.org/docs.html), and the efficiency of these
implementations has been shown in various works [@curtin2013mlpack, @fang2016m3]
using mlpack's benchmarking system [@edel2014automatic].

These algorithms are available via automatically-generated bindings to Python,
R, Go, Julia, and the command-line.  (R, Julia, and Go are new as of mlpack 4.)
Importantly, each of these bindings has a unified interface across different
languages, meaning that, e.g., a model trained in Python can be easily used from
Julia (or C++ or any other language mlpack provides bindings to).  These
bindings are available in each language's package manager, as well as
system-level package managers such as `apt` and `dnf`.  In addition,
ready-to-use Docker containers with the environment fully set up are available
on DockerHub, and an interactive C++ notebook interface via the
[xeus-cling](https://github.com/QuantStack/xeus-cling) project is available on
BinderHub.

Once a user has developed a machine learning workflow in the language of their
choice, deployment is straightforward.  mlpack is now a header-only library, and
depends only on the Armadillo [@sanderson2016armadillo], ensmallen
[@curtin2021ensmallen], and [cereal](https://github.com/USCILab/cereal)
libraries, all of which are header-only.  In most situations, when using C++,
the only linking requirement is to a BLAS/LAPACK implementation (required via
Armadillo).  This significantly eases deployment; a standalone C++ application
with only a BLAS/LAPACK dependency is trivially deployable to all manner of
deployment environments, from standard Linux-based Docker containers to Windows
environments to resource-constrained embedded environments.  To this end,
mlpack's build system now also contains a number of tools for cross-compilation
support, including the ability to easily statically link compiled programs
(important for some deployment environments).

# Major Changes

Below we detail a few of the major changes present in mlpack 4.  For a complete
and exhaustive list (including numerous bugfixes and new techniques), the
`HISTORY.md` file in the code should be consulted.

*Removed dependencies.* In accordance with the vision document [cite], the
majority of work that has gone into the refactoring and redesign of mlpack has
been focused on reducing dependencies and compilation overhead.  This has
motivated the replacement of the [Boost](https://www.boost.org) C++ libraries,
upon which mlpack used to deeply depend, with lightweight replacements including
[cereal](https://github.com/USCILab/cereal) for serialization.  The entire
neural network module was refactored to avoid the use of Boost (almost amounting
to a ground-up rewrite).  This effort was rewarded handsomely: with mlpack 3, a
simple program would often require several GB of RAM just for compilation.
After refactoring and removing dependencies, compilation can sometimes require
just a few hundred MB of RAM, and is often an order of magnitude faster.

*Interactive notebook environments.* Via the
[xeus-cling](https://github.com/QuantStack/xeus-cling) project, mlpack can be
used in a Jupyter notebook environment [@kluyver2016jupyter].  This is
demonstrated
interactively on the [mlpack homepage](https://www.mlpack.org).  A number of
examples of C++ notebooks can be found in the [mlpack examples
repository](https://github.com/mlpack/examples), and these can easily be run on
BinderHub.

*New bindings and availability.* Support for the Julia, Go, and R 
[@bezanson2017julia, @pike2012go, @rcore2022, @parihar2022rmlpack]
languages has been added via mlpack's automatic binding system.  These bindings
can be used by installing mlpack from the language's package manager (`Pkg.jl`,
`go get`, `install.packages('mlpack')`).  In addition, since mlpack's reduced dependency footprint has
significantly simplified the deployment process, mlpack's Python dependencies
are now available for numerous architectures---including Windows---both on PyPI
and in `conda-forge`.

*Cross-compilation support and build system improvements.* As mentioned earlier,
mlpack's CMake configuration now supports easy cross-compilation, for instance
via toolchains such as [buildroot](https://buildroot.org).  By specifying a few
CMake flags, a user may produce a working mlpack setup for a variety of embedded
systems.  This required the implementation of a dependency auto-downloader,
which is capable of downloading [OpenBLAS](https://github.com/xianyi/OpenBLAS)
and compiling (if necessary) for the target architecture.  This auto-downloader
can also be enabled and used for any situation, thus easing installation and
deployment.

# Special Thanks

mlpack is a community-led library, and the software is the product of the hard
work of over 220 individuals (at the time of this writing).  We are indebted to
all of these contributors, as well as all those who have opened issues and bug
reports over the years.  mlpack's development has been supported by Google, via
a decade-long participation the Google Summer of Code program, and also by
NumFOCUS, which fiscally sponsors mlpack.
