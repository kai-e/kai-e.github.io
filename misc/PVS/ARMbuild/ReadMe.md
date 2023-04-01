# PVS Build Instructions for ARM-Based Macs

## Preparation

I have Xcode, its command line utilities, [Emacs for Mac OS X](https://emacsformacosx.com/), and [homebrew](https://brew.sh/) installed.

To use that emacs, set
```
$ export EMACS=/Applications/Emacs.app/Contents/MacOS/Emacs
$ export PVSEMACS=$EMACS
```
If you don't intend to use PVS with the emacs interface then you can do with
```
$ brew install emacs
```

From memory I did (at least):
```
$ brew install llvm sbcl
```
Follow the _Caveats_ at the end.

NB: I do not know whether Apple's clang would be good enough.

Clone both the PVS and the [SBCL](https://www.sbcl.org/) git repositories (say, into `~/src`). 
```
$ cd ~/src
$ git clone git@github.com:SRI-CSL/PVS.git
$ git clone git://git.code.sf.net/p/sbcl/sbcl
```

## Build SBCL

Build and install that SBCL. I used
```
$ cd ~/src/sbcl
$ sh make.sh --prefix=/Users/kaie/opt/sbcl --dynamic-space-size=4Gb
$ sh install.sh
```
(Iirc the `--prefix` didn't work with `~/opt/sbcl`.) Currently that succeeds with
```
//checking for leftover cold-init symbols
Found 5:
(#:*!DELAYED-DEFMETHOD-ARGS* #:!EARLY-GF-NAME #:!BOOTSTRAP-SET-SLOT
 #:!BOOTSTRAP-SLOT-INDEX #:!HAIRY-DATA-VECTOR-REFFER-INIT)
Found 5 fdefns named by uninterned symbols:
(#<SB-KERNEL:FDEFN #:TOKENS> #<SB-KERNEL:FDEFN #:!EARLY-GF-NAME>
 #<SB-KERNEL:FDEFN #:!BOOTSTRAP-SET-SLOT>
 #<SB-KERNEL:FDEFN #:!BOOTSTRAP-SLOT-INDEX>
 #<SB-KERNEL:FDEFN #:!HAIRY-DATA-VECTOR-REFFER-INIT>)
 ```
near the end. Let's hope that that's not fatal.

Install [quicklisp](https://www.quicklisp.org/beta/).

Install PVS dependencies (found via trial and error).
```
$ export SBCLISP_HOME=~/src/sbcl
$ export SBCL_HOME=~/opt/sbcl/lib/sbcl
$ sbcl
* (ql:quickload "BABEL")
* (ql:quickload "CLACK")
* (ql:quickload "WEBSOCKET-DRIVER")
[...install hangs at the end?...C-c and go again with the rest...]
* (ql:quickload "HUNCHENTOOT")
* (ql:quickload "ANAPHORA")
* (ql:quickload "LPARALLEL")
* (exit)
```
And now do something that helps PVS.
```
$ export SBCL_HOME=$SBCLISP_HOME
```
NB: Perhaps the sbcl install step above was redundant because it did not make available that precious `run-sbcl.sh` also missing from the brew-installed version. After this `sbcl` probably won't work in this shell.

## Build PVS

with a few detours to keep track of the current issues.
```
$ cd ../PVS
$ make clean && ./configure && make
[...]
***** make-in-platform /Users/kaie/src/PVS/src/utils/ file_utils 2
debugger invoked on a SIMPLE-ERROR in thread
#<THREAD "main thread" RUNNING {70052B0A73}>:
  Failure in make -C /Users/kaie/src/PVS/src/utils//arm-MacOSX:
output:
clang  -g -O2 -Wall -pedantic -std=gnu99 -mtune=native -mcpu=apple-a14 -c ../file_utils.c -o file_utils.o
clang  -g -O2 -Wall -pedantic -std=gnu99 -mtune=native -mcpu=apple-a14 -c ../utils_table.c -o utils_table.o
/opt/homebrew/Cellar/llvm/14.0.6_1/bin/ld64.lld -dylib -flat_namespace -undefined suppress -arch arm64 -platform_version macos 11.0.0 12.0 -o file_utils.dylib file_utils.o utils_table.o
error output:
make[1]: /opt/homebrew/Cellar/llvm/14.0.6_1/bin/ld64.lld: No such file or directory
make[1]: *** [file_utils.dylib] Error 1
```
Even after confirming that we're using the brew-installed llvm and
```
$ export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
$ export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
```
the same error persists. Why? Because `./configure` asks gcc for its version and `gcc` is from Apple's command line tools! It detects the wrong version, `14.0.6_1` in my case. I do have `16.0.0` installed via homebrew. So let's cheat with something like
```
$ pushd
$ cd /opt/homebrew/Cellar/llvm
$ ln -s 16.0.0 14.0.6_1
$ popd
```
We try again and run into some C errors:
```
$ make
[...]
***** make-in-platform /Users/kaie/src/PVS/src/BDD/ mu 2
debugger invoked on a SIMPLE-ERROR in thread
#<THREAD "main thread" RUNNING {7006270293}>:
  Failure in make -C /Users/kaie/src/PVS/src/BDD//arm-MacOSX:
output:
clang -O2 -g -O2 -Wall -pedantic -std=gnu99 -mtune=native -mcpu=apple-a14 -I/usr/include -I../bdd/src -I../bdd/utils -I../mu/src -c ../bdd_table.c -o bdd_table.o
clang -O2 -g -O2 -Wall -pedantic -std=gnu99 -mtune=native -mcpu=apple-a14 -I/usr/include -I../bdd/src -I../bdd/utils -I../mu/src -c ../mu_table.c -o mu_table.o
error output:
In file included from ../bdd_table.c:2:
../bdd/utils/hash.h:74:27: warning: a function declaration without a prototype is deprecated in all versions of C [-Wstrict-prototypes]
  void (*rehash_function) ();   /* function called when non-NULL */
                          ^
                           void
../bdd_table.c:101:4: warning: void function 'bdd___bdd_print' should not return void expression [-Wpedantic]
  {return bdd_print (fp, f, s);}
   ^      ~~~~~~~~~~~~~~~~~~~~
../bdd_table.c:103:29: warning: void function 'bdd___bdd_quit' should not return void expression [-Wpedantic]
void bdd___bdd_quit (void) {return bdd_quit ();}
                            ^      ~~~~~~~~~~~
../bdd_table.c:169:4: warning: void function 'bdd___bdd_free_vec' should not return void expression [-Wpedantic]
  {return bdd_free_vec (f_vec, size);}
   ^      ~~~~~~~~~~~~~~~~~~~~~~~~~~
4 warnings generated.
In file included from ../mu_table.c:2:
In file included from ../mu_interface.h:4:
In file included from ../mu/src/mu.h:17:
../bdd/utils/hash.h:74:27: warning: a function declaration without a prototype is deprecated in all versions of C [-Wstrict-prototypes]
  void (*rehash_function) ();   /* function called when non-NULL */
                          ^
                           void
In file included from ../mu_table.c:2:
../mu_interface.h:28:24: warning: a function declaration without a prototype is deprecated in all versions of C [-Wstrict-prototypes]
extern LIST empty_list ();
                       ^
                        void
../mu_table.c:83:34: error: parameter 'bdd_idx' was not declared, defaults to 'int'; ISO C99 and later do not support implicit int [-Wimplicit-int]
char* mu___get_mu_bool_var_name (bdd_idx)
                                 ^
../mu_table.c:83:7: warning: a function definition without a prototype is deprecated in all versions of C and is not supported in C2x [-Wdeprecated-non-prototype]
char* mu___get_mu_bool_var_name (bdd_idx)
      ^
../mu_table.c:89:22: warning: a function declaration without a prototype is deprecated in all versions of C [-Wstrict-prototypes]
LIST mu___empty_list () {return (LIST) empty_list ();}
                     ^
                      void
4 warnings and 1 error generated.
make[1]: *** [mu_table.o] Error 1

Type HELP for debugger help, or (SB-EXT:EXIT) to exit from SBCL.
[...]
```
We can fix these easily with this [patch](0001-Fix-C-errors-for-the-SBCL-based-ARM-Mac-build.patch).

After another `make` -- that should now succeed -- we're ready to

## Test

```
$ ./pvs -q # could append `-load-after ~/.emacs`
```

