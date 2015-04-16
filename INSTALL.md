Installing Scholdoc
===================

This document describe how to completely install Scholdoc from source. The
instructions are largely adapted from Pandoc's installation instructions.

If you are installing the development version from Github, the procedure is
identical to Pandoc:

https://github.com/jgm/pandoc/wiki/Installing-the-development-version-of-pandoc

However, Scholdoc relies on its own fork of `pandoc-types`, `texmath`, and
`pandoc-citeproc`. The reposity for all these packages can be found in this
post:

http://forum.scholarlymarkdown.com/t/scholdoc-repository-information/ 


Setting up a Haskell build chain
--------------------------------

Scholdoc is written in pure Haskell. It requires the [GHC] compiler and the
[cabal-install] build system. The easiest way to get it on all platforms is by
installing the [Haskell platform] for your operating system. Please make sure
you are using GHC version 7.4 or above. (unless you are using [Homebrew] with
OSX or Ubuntu >12.04, see below).

### Using [Homebrew]

OSX users running [Homebrew] can easily get the prerequisites by running

    brew update
    brew install ghc cabal-install

You can also install Scholdoc directly from the offical [Homebrew][Homebrew] [tap][homebrew-scholdoc] by running

    brew tap timtylin/scholdoc
    brew update
    brew install scholdoc scholdoc-citeproc

### Using the `hvr/ghc` PPA on Ubuntu (12.04 or later)

Herbert V. Riedel have conveniently provided a [PPA of pre-compiled GHC and
Cabal][hvr-PPA] for recent Ubuntu systems. Here's an example of how to get
recommended versions of [GHC] and [cabal-install] using `apt-get`

    sudo add-apt-repository ppa:hvr/ghc
    sudo apt-get update && apt-get install ghc-7.8.3 cabal-install-1.20


Quick install (stable release from Hackage)
-------------------------------------------

1.  Install the [Haskell platform].  This will give you [GHC] and the 
    [cabal-install] build tool.

2.  Update your package database:

        cabal update

3.  Use `cabal` to install Scholdoc and its dependencies:

        cabal install scholdoc

    This procedure will install the released version of Scholdoc, which will be
    downloaded automatically from HackageDB. It will also install
    scholdoc-citeproc
    
Quick install (your own version)
--------------------------------

If you want to install a modified or development version
of Scholdoc instead, switch to the source directory.

1.  Install the [Haskell platform].  This will give you [GHC] and the 
    [cabal-install] build tool.

2.  Update your package database:

        cabal update

3.  Use the provided makefile to trigger the right install instructions for `cabal`:

        make deps
        make install

    **Note:** If you obtained the source from the git repository (rather than a
    release tarball), you'll need to do

        git submodule update --init

    to fetch the contents of `data/templates` before `cabal install`.

4.  Make sure the `$CABALDIR/bin` directory is in your path.  You should now be 
    able to run `scholdoc`:

        scholdoc --help

    [Not sure where `$CABALDIR` is?](http://www.haskell.org/haskellwiki/Cabal-Install#The_cabal-install_configuration_file)

Installing Scholdoc-citeproc
----------------------------

If you want to process citations with Scholdoc, you will also need to install a
separate package, `scholdoc-citeproc`. This can be installed using cabal:

    cabal install scholdoc-citeproc

### Locale-sensitive unicode collation

By default `scholdoc-citeproc` uses the "i;unicode-casemap" method to sort
bibliography entries (RFC 5051). If you would like to use the locale-sensitive
unicode collation algorithm instead, specify the `unicode_collation` flag:

   cabal install scholdoc-citeproc -funicode_collation

Note that this requires the `text-icu` library, which in turn depends on the C
library `icu4c`. Installation directions vary by platform. Here is how it might
work on OSX with homebrew:

    brew install icu4c
    cabal install --extra-lib-dirs=/usr/local/Cellar/icu4c/51.1/lib \
      --extra-include-dirs=/usr/local/Cellar/icu4c/51.1/include \
      -funicode_collation text-icu scholdoc-citeproc


Custom install
--------------

This is a step-by-step procedure that offers maximal control over the build and
installation. Most users should use the quick install, but this information may
be of use to packagers. For more details, see the [Cabal User's Guide]. These
instructions assume that the Scholdoc source directory is your working
directory.

1.  Install dependencies: in addition to the [Haskell platform], you will need
    a number of additional libraries. You can install them all with

        cabal update
        cabal install --only-dependencies

2.  Configure:

        cabal configure --prefix=DIR --bindir=DIR --libdir=DIR \
          --datadir=DIR --libsubdir=DIR --datasubdir=DIR --docdir=DIR \
          --htmldir=DIR --program-prefix=PREFIX --program-suffix=SUFFIX \
          --mandir=DIR --flags=FLAGSPEC

    All of the options have sensible defaults that can be overridden as needed.

    `FLAGSPEC` is a list of Cabal configuration flags, optionally preceded by a
    `-` (to force the flag to `false`), and separated by spaces. Scholdoc's
    flags include:

    - `embed_data_files`: embed all data files into the binary (default no).
      This is helpful if you want to create a relocatable binary. Note: if this
      option is selected, you need to install the `hsb2hs` preprocessor:

          cabal install hsb2hs

    - `https`: enable support for downloading resources over https (using the
      `http-client` and `http-client-tls` libraries).
    
    - `tryscholdoc`: builds a CGI executable that can be served to parse
      ScholarlMarkdown text over the internet. See the contents of the
      `tryscholdoc` directory for more details.

3.  Build:

        cabal build

4.  Build API documentation:

        cabal haddock --html-location=URL --hyperlink-source

5.  Copy the files:

        cabal copy --destdir=PATH

    The default destdir is `/`.

6.  Register Scholdoc as a GHC package:

        cabal register

    Package managers may want to use the `--gen-script` option to generate a
    script that can be run to register the package at install time.

Creating a relocatable binary
-----------------------------

It is possible to compile Scholdoc such that the data files Scholdoc uses are
embedded in the binary. The resulting binary can be run from any directory and
is completely self-contained.

    cabal install hsb2hs  # a required build tool
    cabal install --flags="embed_data_files" citeproc-hs
    cabal configure --flags="embed_data_files"
    cabal build

You can find the Scholdoc executable in `dist/build/scholdoc`. Copy this
wherever you please.

If you use the included `Makefile` commands, then embedding the binary files is
the default behavior.

Running tests
-------------

Scholdoc comes with an automated test suite integrated to cabal. The fastest
way to access the tests is by the Makefile commands.

To build the tests:

    make deps
    make quick

To run the tests:

    make test

To run particular tests (pattern-matching on their names), use `cabal test`
with the `-t` option:

    cabal test --test-options='-t markdown'

If you add a new feature to Scholdoc, please add tests as well, following the
pattern of the existing tests. The test suite code is in
`tests/test-scholdoc.hs`. It is probably easiest to add/modify data files to
the `tests` directory (and change `tests/Tests/Old.hs` if addiitons are made).
For small, detailed behavior and messy edge cases, it is better to modify the
tests under the `tests/Tests` hierarchy corresponding to the Scholdoc module
you are changing.

Running benchmarks
------------------

To build the benchmarks:

    make deps
    make full

To run the benchmarks:

    make bench

To change the time allowed for each reader/writer benchmark:

    cabal bench --benchmark-options='--time-limit 1.0'

To run just the markdown benchmarks:

    cabal bench --benchmark-options='markdown'


[GHC]: http://www.haskell.org/ghc/
[Haskell platform]: http://hackage.haskell.org/platform/
[cabal-install]: http://hackage.haskell.org/trac/hackage/wiki/CabalInstall
[zip-archive]: http://hackage.haskell.org/package/zip-archive
[highlighting-kate]: http://hackage.haskell.org/package/highlighting-kate
[blaze-html]: http://hackage.haskell.org/package/blaze-html
[Cabal User's Guide]: http://www.haskell.org/cabal/release/latest/doc/users-guide/builders.html#setup-configure-paths
[Homebrew]: http://brew.sh
[hvr-PPA]: https://launchpad.net/~hvr/+archive/ubuntu/ghc
[homebrew-scholdoc]: https://github.com/timtylin/homebrew-scholdoc/