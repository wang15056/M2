# Location of `decompose` Function Definition and Implementation

## Summary

The `decompose` function in Macaulay2 is used to compute the minimal prime decomposition of an ideal. This document describes where this function is declared and implemented in the M2 repository.

## Declaration

**File:** `/M2/Macaulay2/m2/shared.m2`  
**Line:** 19

```macaulay2
decompose = method(Options => true)
```

This declares `decompose` as a method that accepts options.

## Implementation

### Primary Implementation for Ideal Type

**File:** `/M2/Macaulay2/packages/MinimalPrimes.m2`  
**Line:** 195

```macaulay2
decompose Ideal := List => options minimalPrimes >> opts -> I -> minprimesHelper(I, (minimalPrimes, Ideal), opts)
```

This implementation shows that:
- `decompose` for an `Ideal` returns a `List`
- It inherits options from `minimalPrimes`
- It delegates to the `minprimesHelper` function

### Helper Function: minprimesHelper

**File:** `/M2/Macaulay2/packages/MinimalPrimes.m2`  
**Lines:** 219-246

The `minprimesHelper` function performs the actual computation:

```macaulay2
minprimesHelper = (I, key, opts) -> (
    if I == 1 then return {};
    J := first flattenRing I;
    if J == 0 then return {I};
    -- TODO: make presentation work for ZZ, then move this line
    if ring I === ZZ then return ideal \ first \ toList factor (trim I)_0;
    S := ring presentation ring J;

    strategy := opts.Strategy;
    doTrim := if opts.MinimalGenerators then trim else identity;

    codimLimit := min(opts.CodimensionLimit, dim S, numgens J);
    doLimit := L -> select(L, P -> codim(P, Generic => true) <= codimLimit);
    opts = opts ++ { CodimensionLimit => codimLimit };

    -- this logic determines what strategies will be used
    computation := (opts, container) -> runHooks(key, (opts, I), Strategy => opts.Strategy);

    -- this is the logic for caching partial minimal primes computations
    container := fetchComputation(MinimalPrimesComputation, I, new MinimalPrimesContext from I);

    -- the actual computation of minimal primes occurs here
    L := (cacheComputation(opts, container)) computation;

    if L =!= null then doLimit \\ doTrim \ L else if strategy === null
    then error("no applicable strategy for ", toString key)
    else error("assumptions for minimalPrimes strategy ", toString strategy, " are not met"))
```

### Key Implementation Details

1. **Package:** The implementation is in the `MinimalPrimes` package (MinimalPrimes.m2)

2. **Strategies:** Multiple strategies are available for computing minimal primes (Lines 259-314):
   - `"Legacy"` - Traditional algorithm for ideals over QQ or ZZ/p
   - `"NoBirational"` - Algorithm without birational strategies (default for most cases)
   - `"Birational"` - Algorithm with birational strategies
   - `Hybrid` - Advanced custom strategies
   - `Monomial` - Specialized algorithm for monomial ideals

3. **Algorithm Strategy Components** (Lines 252-257):
   ```macaulay2
   strat0 = ({Linear, DecomposeMonomials}, infinity)
   strat1 = ({Linear, DecomposeMonomials, (Factorization, 3)}, infinity)
   BirationalStrat = ({strat1, (Birational, infinity)}, infinity)
   NoBirationalStrat = strat1
   stratEnd = {(IndependentSet, infinity), CharacteristicSets}
   ```

   Strategy components include:
   - `Linear` - Handle linear components
   - `DecomposeMonomials` - Decompose monomial ideals
   - `Factorization` - Use factorization
   - `Birational` - Use birational transformations
   - `IndependentSet` - Use independent sets
   - `CharacteristicSets` - Use characteristic sets

4. **Relationship to minimalPrimes:** 
   - `decompose` is essentially an alias for `minimalPrimes`
   - Both use the same helper function `minprimesHelper`
   - Both share the same options and strategies
   - Line 195 shows: `decompose Ideal := List => options minimalPrimes >> opts -> ...`

## Usage in the Problem Statement

In the provided code example:
```macaulay2
dec = decompose(IdualX+IdualZ);
```

This call:
1. Invokes `decompose` on an ideal (IdualX+IdualZ)
2. Returns a list of ideals representing the irreducible components
3. Uses the default strategy determined by the MinimalPrimes package

### Complete Example from Problem Statement

```macaulay2
restart
n = 3;
d = 2;
N = binomial(n-1+d,d);
KK = ZZ/101;

-- setup code omitted for brevity ...

-- Computing intersection of dual varieties
(codim IdualX, degree IdualX)   -- (1,3)
(codim IdualZ, degree IdualZ)   -- (1,6)
(codim IdualXZ, degree IdualXZ) -- (2,6)

-- Decompose the intersection into irreducible components
dec = decompose(IdualX+IdualZ);
#dec                            -- returns 2, indicating two irreducible components
dec#0 == IdualXZ                -- true, one component is the relative dual variety
(codim(dec#1), degree(dec#1))   -- (2,6), second component also has same dimension
```

This demonstrates how `decompose` is used to find the irreducible components of the intersection of two algebraic varieties (specifically, dual varieties in this geometric context).

## Available Options

From the `minimalPrimes` method definition (lines 181-190), `decompose` accepts these options:

```macaulay2
- Verbosity              => 0          -- control output verbosity
- Strategy               => null       -- which strategy to use (auto-selected if null)
- CodimensionLimit       => infinity   -- only find components with codim ≤ this bound
- MinimalGenerators      => true       -- whether to trim the output ideals
- "CheckPrimeOnly"       => false      -- internal option
- "SquarefreeFactorSize" => 1          -- internal option for factorization
```

Example with options:
```macaulay2
decompose(I, Strategy => "Legacy", Verbosity => 1)
decompose(I, CodimensionLimit => 3)  -- only find components of codimension ≤ 3
```

## Package Information

**Package Name:** MinimalPrimes  
**Version:** 0.10  
**Date:** November 12, 2020  
**Headline:** minimal primes and radical routines for ideals

**Authors:**
- Frank Moore (moorewf@wfu.edu)
- Mike Stillman (mike@math.cornell.edu)
- Franziska Hinkelmann
- Justin Chen (justin.chen@math.gatech.edu)
- Mahrud Sayrafi (mahrud@umn.edu)

## Documentation

**File:** `/M2/Macaulay2/packages/Macaulay2Doc/shared.m2`  
**Line:** 22

```macaulay2
document { Key => decompose, methodstr, SeeAlso => { "MinimalPrimes::MinimalPrimes" } }
```

The documentation for `decompose` refers users to the MinimalPrimes package documentation.

## Related Files

- `/M2/Macaulay2/m2/shared.m2` - Method declaration (line 19)
- `/M2/Macaulay2/packages/MinimalPrimes.m2` - Main implementation (line 195)
- `/M2/Macaulay2/packages/MinimalPrimes/` - Additional test files and implementations
- `/M2/Macaulay2/packages/Macaulay2Doc/shared.m2` - Documentation (line 22)

## See Also

- `minimalPrimes` - Related function that computes minimal primes (essentially the same as decompose)
- `radical` - Computes the radical of an ideal (intersection of all minimal primes)
