# Adding `NASALIB`

```
git clone git@github.com:nasa/pvslib.git nasalib 
export PVS_LIBRARY_PATH="$PWD/nasalib"
cd nasalib
./install-scripts
```
This appends an elisp-snippet to `~/.pvsemacs` that doesn't work in my fairly new emacs. 
**Fix:** Change `loop` in that to `cl-loop`.

Currently, the changes to proofs to help PVS 8.0 aren't merged yet, so we pull them from Sam's clone:
```
brew install gh # only once, if github's CLI isn't installed
gh auth login   # only if the next step fails with auth problems
gh pr checkout 13
```
