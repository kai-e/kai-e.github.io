# Adding `NASALIB`

```console
$ cd ~/src/PVS # or wherever the PVS sources are
$ git clone git@github.com:nasa/pvslib.git nasalib 
$ export PVS_LIBRARY_PATH="$PWD/nasalib"
$ export PVS_DIR=$PWD
$ cd nasalib
$ ./install-scripts
```
This appends an elisp-snippet to `~/.pvsemacs` that doesn't work in my fairly new emacs. 
**Fix:** Change `loop` in that to `cl-loop`.

Currently, the changes to proofs to help PVS 8.0 aren't merged yet, so we pull them from Sam's clone:
```console
$ brew install gh # only once, if github's CLI isn't installed
$ gh auth login   # only if the next step fails with auth problems
$ gh pr checkout 13
```

## MetiTarski & Z3

Currently, the nasalib contains binaries for `metit` and `z3` for x86 (linux and Mac OS X). Maybe this is ok via Rosetta II but hey, we want this to be pure.

```console
$ brew install z3 polyml # singular
$ cd ~/src
$ git clone https://bitbucket.org/lcpaulson/metitarski-git.git
$ cd metitarski-git
$ ./configure  # add your options here, I had --prefix=~/opt/metit CC=clang LDFLAGS="-L/opt/homebrew/opt/llvm/lib -L/opt/homebrew/lib"
$ make && make install
```
Add
```bash
export Z3_NONLIN=/opt/homebrew/bin/z3
export METITARSKI=~/src/metitarski-git # or perhaps where you installed it?
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$METITARSKI
export TPTP=$METITARSKI/tptp
```
to `~/.profile` or equivalent.

In dryruns of `metit` I had to give it the `--z3` option to avoid running into Errors. This can be enforced by injecting `--z3`  in line 363 of `MetiTarski/metit.lisp`. The nasalib scripts (`prove-all`, `provethem`, `proveit` etc.) did not work when given `--disable-oracles` as it's the default set in line 5 of `nasalib/nasalib.all`. So remove that for now.

## QEPCAD (in theory, have yet to try this)

is an alternative external arithmetic decision procedure. To use [QEPCAD](https://www.usna.edu/Users/cs/wcbrown/qepcad/B/QEPCAD.html), download sources
```console
$ curl --output-dir ~/src --remote-name-all https://www.usna.edu/Users/cs/wcbrown/qepcad/INSTALL/qepcad-B.1.74.tgz https://www.usna.edu/Users/cs/wcbrown/qepcad/INSTALL/saclib2.2.8.tgz
```
and compile by following the [instructions](https://www.usna.edu/Users/cs/wcbrown/qepcad/INSTALL/IQ.html).


