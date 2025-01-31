# Tornado


Tornado is a compiler producing masked bitsliced implementations proven secure in the bit/register probing model. It was introduced in the following publication

[Tornado: Automatic Generation of Probing-Secure Masked Bitsliced Implementations](https://eprint.iacr.org/2020/506)

by 

* Sonia Belaïd 
* Pierre-Evariste Dagand 
* Darius Mercadier
* Matthieu Rivain
* Raphaël Wintersdorff

## About

Tornado is composed of (a modified version of) [__usuba__](https://github.com/DadaIsCrazy/usuba) and __tightPROVE+__ (an extension of [tightPROVE](https://github.com/CryptoExperts/tightPROVE)).


Respective documentation for each tools separately can be found in
`src/usuba/Readme.md` and `src/tightPROVEp/manual.txt`.

What follows is the documentation for their combination. If you just
want to use the tool, we recommand that you first read
`src/usuba/Readme.md`, which explains how to use `Usuba`. To then use
`Tornado`, just add the flags `-tp -ua-masked` (and optionally
`-light-inline`) to your `usubac` compile lines.

## Quick Installation Guide with Docker


1. Our artefact (the Tornado compiler & supporting benchmarks) is
   distributed through a Docker container. The first step is thus to
   install Docker on your machine:

   On Debian & Ubuntu: `sudo apt-get install docker.io`
                    or `sudo apt-get install docker-ce`
   On Arch:            `sudo pacman -S docker`
   On other platforms, please refer to:

       https://docs.docker.com/install/

2. To make our container, run:

       make

3. To interact with the toolchain, run:

       docker run -ti --hostname dadaubuntu tornado /bin/bash
       
    which will give you a shell inside the container

## Long Installation Guide

### TightPROVE+

You need to install SageMath 8.9 or later. This can be done either via
your distribution's packages, or from source (or precompiled binaries)
on [www.sagemath.org](https://www.sagemath.org/download-linux.html).


### Usuba

`src/usuba/Readme.md` explains how to install Usuba. In a nutshell:

    cd src/usuba
    ./install_deps.pl
    ./configure.pl # See bellow

Note that before running `configure.pl`, you should edit
`src/usuba/config.json` with the path of you `sage` binary. See
`src/usuba/config.json.help` for more details.


### Tornado

Once you have installed SageMath as well as Usuba's dependencies, you
can run `src/make`. This will compile Usuba with the correct paths for
sage and tightPROVE+.

This will create a binary named `tornado` in the current folder. This
binary can be used like `usubac`.


### Checking that installation was successful

Once you have installed TightPROVE+, Usuba, and Tornado, you can test
that your tornado build works by running:

    ./simple_test/run.pl
    
This will run `tornado` on `simple_test/ace_f.ua`, which an Usuba
implementation of the `f` function from the cipher ACE. `run.pl` will
try to compile this Usuba code using Tornado, and will make sure that
the result contains a refresh, as it should.

If this script prints `Tornado seems to work.`, then you are good to
go, and you may start using Tornado.

## Using Tornado

`tornado` can be used in the same fashion as `usubac`, with a few more
flags. Mainly:

   - `-tp`: calls tightPROVE+ to insert refreshes as needed.

   - `-ua-masked`: masks the code generated by `usubac` within
     `usubac`. Loop fusion is performed, as well as
     constant-propagation within multiplications.

   - `-masked`: masks the code generated by `usubac` when emitting `C`
     code. No optmizations related to masking are performed.  We
     recommand using this flag rather than `-ua-masked` for debugging,
     but you should switch to `-ua-masked` in production for better
     performances.
     
   - `-light-inline`: Usuba's previous behaviour was to perform very
     aggressive inlining, as it was targetting high-end intel CPUs,
     with large amount of RAM. However, in our setting, it is better
     to keep binary size low and avoid inlining too many
     functions. Using this flag, only functions explicitely marked
     with `_inline` are inlined.

Furthermore, we highly recommand that you pass on either `-B` or `-V`
to `tornado`, depending on whether you compile your source to
respectively bitsliced or nsliced code.

The `C` codes generated will use a macro called `MASKING_ORDER` that
will not be be defined. In order to compile them at a given order,
you'll need of either `#define MASKING_ORDER <n>` or to pass
`-DMASKING_ORDER=<n>` to your C compiler.

For more examples on how to use `tornado`, we refer to the files
`src/usuba/gen_nist_benchs.sh`, and `src/usuba/gen_nist_benchs_masked.sh`.

Be aware that compiling a cipher with `-tp` will take some time in all
likelyhood. However, since there is a cache, subsequent compilation
should be faster. **If you are in a hurry**, you can always omit the
`-tp` flag, which means that tornado will not call tightPROVE+ to
insert additional refreshes. Be careful however, since the resulting
code might be vulnerable.


You will find a lot of cipher implementations in `src/usuba/samples/usuba`.

## Example

Take for instance the cipher `Ace`, whose `usuba` implementation is to
be found in `src/usuba/samples/usuba/ace.ua`. As it is, it's currently
vulnerable due to a lack of refreshing of `x` inside its function `f`.

You can compile it to bitslice and vslice masked code using:

    ./tornado -B -light-inline -tp -ua-masked -o ace_bitslice.c usuba/samples/usuba/ace.ua
    ./tornado -V -light-inline -tp -ua-masked -o ace_nslice.c usuba/samples/usuba/ace.ua

This will produce two `C` files. Inspecting them will reveal that the
`f` function of the nsliced implementation contains an refresh, which
wasn't in the original source. (the bitsliced implementation doesn't
need this refresh)

## More examples

You can find a lot of ciphers in Usuba inside `src/usuba/samples/usuba`.

More details can be found about the ciphers from the NIST lightweight
cipher competition in `src/usuba/nist/`. The structure of this directory
is as follows:

  - `src/usuba/nist/<cipher>/usuba/ua` contains basic correctness-checking
    for Usuba generated code: the `main.c` run usuba-generated
    versions of the ciphers as well as reference implementations, and
    make sure the results are the same. (folders `ua_masked` serve the
    same purposes but for masked implementations)

  - `src/usuba/nist/<cipher>/usuba/bench` contains C codes generated from
    Usuba implementations as well as reference implementations,
    destined to be benchmarked. On an Intel CPU, you can use the
    scripts `src/usuba/bench_nist.pl` and `src/usuba/bench_nist_masked.pl` to
    benchmark those implementations.

## License 

* Usuba is under [MIT license](https://opensource.org/licenses/MIT)
* tightPROVE+ is under [GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html)