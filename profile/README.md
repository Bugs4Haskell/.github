# Bugs4Haskell

## How to contribute

1. Create a repository with the following structure (or use the skeleton):

```
.
├── README.md       
├── foo.cabal
├── src                -- The code of your case study
│   ├── Foo.hs        
│   ├── ...      
└── test               -- A simple runnable test suite consisting of:
    ├── Main.hs        -- Entry point
    ├── Arbitrary.hs   -- Arbitrary instances
    ├── Spec.hs        -- QuickCheck properties
    └── Bench.hs       -- Benchmarking functions
```

2. Inject bugs using CPP flags and declare them in your .cabal file:

### In `src/Foo.hs`:

```
foo :: Int
#if defined(BUG_FOO_1)
foo = 24
#else 
foo = 42
#endif
```

### In `foo.cabal`:

```
cabal-version:  2.2
name:           stack
version:        0.1.0
build-type:     Simple

flag BUG_FOO_1
  default: False

common bugs
  if flag(BUG_FOO_1)
    cpp-options: -BUG_FOO_1

library
  import: bugs
  exposed-modules:
    Foo
  hs-source-dirs:
    src
  default-language: Haskell2010
  default-extensions: CPP
  build-depends: base, ...

executable tests
  import: bugs
  main-is: Main.hs
  hs-source-dirs: test 
  other-modules:
    Arbitrary
    Spec
    Bench     
  default-language: Haskell2010
  default-extensions: CPP
  build-depends: foo, ...
 ```
 
### In `test/Main.hs`:

```
module Main where

import Spec
import Bench

----------------------------------------
-- Variants

variant :: String
#if defined(BUG_FOO_1)
variant = "BUG_FOO_1"
#else
variant = "VANILLA"
#endif

----------------------------------------
-- Main

main :: IO ()
main = forM_ benchmarks (runBenchmark variant)
```

3. Write QuickCheck properties and create executable benchmarks:

### In `test/Spec.hs`:

```
module Spec where

import Test.QuickCheck

import Arbitrary()
import Bench

----------------------------------------
-- Benchmarks

qc_args :: Args
qc_args = stdArgs { maxSize = 10, maxSuccess = 100000000, maxDiscardRatio = 1000 } 

benchmarks :: [Benchmark]
benchmarks =
  [ ("prop_foo", benchQuickCheck qc_args prop_foo)
  ]

----------------------------------------
-- Properties

prop_foo :: Foo -> Property
prop_foo = ...
```
