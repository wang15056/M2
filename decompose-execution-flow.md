# Code Execution Flow for `decompose` Function

This document traces the execution path when calling `decompose` on an ideal in Macaulay2.

## Call Stack

When you call `decompose(I)` where `I` is an ideal:

```
1. decompose(I)
   └─> Declared at: M2/Macaulay2/m2/shared.m2:19
   
2. decompose Ideal implementation
   └─> Defined at: M2/Macaulay2/packages/MinimalPrimes.m2:195
   └─> Inherits options from minimalPrimes
   └─> Calls: minprimesHelper(I, (minimalPrimes, Ideal), opts)
   
3. minprimesHelper(I, key, opts)
   └─> Defined at: M2/Macaulay2/packages/MinimalPrimes.m2:219-246
   └─> Main logic:
       a. Handle special cases (I==1, I==0, ring I === ZZ)
       b. Flatten the ring and prepare options
       c. Set up computation with runHooks
       d. Cache and fetch partial computations
       e. Execute strategy-based algorithm
   
4. runHooks(key, (opts, I), Strategy => strategy)
   └─> Executes one of the registered strategies:
   
   Strategy Options (defined at lines 259-314):
   
   a. "Legacy" (line 260-271)
      └─> Calls: legacyMinimalPrimes(I)
      └─> For ideals over QQ or ZZ/p
      
   b. "NoBirational" (line 273-281)
      └─> Calls: minprimesWithStrategy(I, strategy=NoBirationalStrat, ...)
      └─> Uses Linear, DecomposeMonomials, and Factorization
      
   c. "Birational" (line 283-291)
      └─> Calls: minprimesWithStrategy(I, strategy=BirationalStrat, ...)
      └─> Uses all strategies including Birational transformations
      
   d. Hybrid (line 293-302)
      └─> Calls: minprimesWithStrategy(I, strategy=custom, ...)
      └─> For custom strategy combinations
      
   e. Monomial (line 304-313)
      └─> For monomial ideals
      └─> Uses: dual radical monomialIdeal I
```

## Strategy Components

Each strategy is built from these components (lines 252-257):

```macaulay2
strat0 = ({Linear, DecomposeMonomials}, infinity)
strat1 = ({Linear, DecomposeMonomials, (Factorization, 3)}, infinity)
BirationalStrat = ({strat1, (Birational, infinity)}, infinity)
NoBirationalStrat = strat1
stratEnd = {(IndependentSet, infinity), CharacteristicSets}
```

### What each component does:

- **Linear**: Splits ideals based on linear polynomials
- **DecomposeMonomials**: Handles monomial ideal components
- **Factorization**: Uses polynomial factorization to split components
- **Birational**: Uses birational transformations (more powerful, potentially slower)
- **IndependentSet**: Works with independent variable sets
- **CharacteristicSets**: Uses characteristic set decomposition

## Return Value

The function returns a `List` of `Ideal` objects, where each ideal represents one irreducible component of the input ideal's decomposition.

## Caching

Results are cached in:
- `I.cache.decompose` or within `MinimalPrimesComputation` objects
- Allows partial results to be reused if computation is interrupted or re-run

## Example Trace

For the problem statement code:
```macaulay2
dec = decompose(IdualX+IdualZ);
```

Execution path:
1. `decompose` method called with ideal `(IdualX+IdualZ)`
2. Routes to `decompose Ideal` implementation
3. Calls `minprimesHelper` with the ideal and options
4. Strategy auto-selected (likely "NoBirational" or "Birational")
5. Computes minimal primes using the selected algorithm
6. Returns list of 2 ideals (the irreducible components)
7. Result: `dec` is a list where:
   - `dec#0` is one component (equal to `IdualXZ`)
   - `dec#1` is the second component

## Files to Read for Deep Understanding

In order of importance:

1. **M2/Macaulay2/packages/MinimalPrimes.m2**
   - Lines 195: Main implementation
   - Lines 219-246: Helper function
   - Lines 259-314: Strategy implementations
   - Lines 181-190: Available options

2. **M2/Macaulay2/m2/shared.m2**
   - Line 19: Method declaration

3. **M2/Macaulay2/packages/MinimalPrimes/...** 
   - Additional strategy implementations and helper files

## Related Functions

- `minimalPrimes`: Identical to `decompose` (both call same helper)
- `radical`: Computes the intersection of all minimal primes
- `isPrime`: Checks if an ideal is prime
- `minors`: Used in conormal variety computations (as in problem statement)
