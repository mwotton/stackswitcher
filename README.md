# stackswitcher - stop rebuilding the world

## why?

stack does a pretty good job in general of caching built packages. It isn't quite so successful at caching built local products, for a number of reasons:

- profiling builds are done by default in the same place
- switching git branches can change enough that almost everything needs to be rebuilt.

## what doesn't work?

```
alias stack='mkdir -p .stack-work; export STACK_REF="ref-$(git rev-parse --abbrev-ref HEAD)"; (cd .stack-work; if [ -d ref-master ] && [ ! -d $STACK_REF ]; then cp -r ref-master $STACK_REF; fi); stack --work-dir .stack-work/$STACK_REF'
```

https://twitter.com/tazjin/status/999056299596877825

This is pretty close - unfortunately, because there are absolute paths in the stack model, you can't move .stack-work directories around and expect them to work in the new location.

## what does work

- create a directory to hold a per-branch .stack-work
- initialise as .stack-work in the root
- add a git hook to move the current .stack-work into the holding directory on branch change.
  - this is a _little_ more complicated than it seems: we also have a .stack-work for each package directory, for reasons to do with a Cabal Windows bug (https://github.com/commercialhaskell/stack/issues/1178#issuecomment-162255770) - we need to cache these too.
  - should also move the equivalent .stack-work-profiling.
- add an alias (pstack) that uses .stack-work-profiling in order not to trash what's there when you want to profile.


## Bugs/TODO

No bugs yet, because no code!

TODO: think about the approach to profiling a little more carefully. arguably we could get the same effect by just opening up a new branch.
