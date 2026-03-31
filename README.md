# harmonic-entropy

Compute harmonic entropy of musical intervals.

## Installation

```bash
npm i harmonic-entropy
```

## Documentation

- API docs (TypeDoc): <https://xenharmonic-devs.github.io/harmonic-entropy>
- Generate docs locally:

```bash
npm run doc
```

## What this package provides

This package exposes:

- `precalculateRatios(options)`: precomputes rational pairs used by the entropy algorithm.
- `harmonicEntropy(options, ratios)`: computes a tabulated entropy curve as `[cents, entropy]` pairs.
- `EntropyCalculator`: convenient class for evaluating intervals (`ofCents`, `ofFraction`) with cached tables.

## Options reference

`HarmonicEntropyOptions` fields (all optional):

- `N`: max rational complexity.
  - default for `series: 'tenney'`: `10000`
  - default for `series: 'farey'`: `1000`
- `s`: Gaussian frequency deviation (default `0.01`)
- `a`: Rényi order (default `1`, evaluated numerically near Shannon entropy)
- `series`: `'tenney' | 'farey'` (default `'tenney'`)
- `minCents`: lower tabulation bound (default `0`)
- `maxCents`: upper tabulation bound (default `2400`)
- `res`: tabulation step in cents (default `1`)
- `normalize`: divide by Hartley entropy term (default `false`)

## Example: entropy table for a range

```ts
import {
  type HarmonicEntropyOptions,
  harmonicEntropy,
  precalculateRatios,
} from 'harmonic-entropy';

const options: HarmonicEntropyOptions = {
  N: 10000,
  s: 0.01,
  a: 1,
  series: 'tenney',
  minCents: 0,
  maxCents: 2400,
  res: 1,
  normalize: false,
};

const ratios = precalculateRatios(options);
const table = harmonicEntropy(options, ratios);

console.log(table[0]); // [0, ...]
console.log(table[table.length - 1]); // [2400, ...]
```

## Example: evaluate individual intervals

```ts
import {EntropyCalculator} from 'harmonic-entropy';

const entropy = new EntropyCalculator({maxCents: 1200});

// Evaluate by cents.
const perfectFifth = entropy.ofCents(700);

// Evaluate by ratio.
const pureFifth = entropy.ofFraction('3/2');

// Numeric input is interpreted as a frequency ratio.
const sameFifth = entropy.ofFraction(3 / 2);

console.log({perfectFifth, pureFifth, sameFifth});
```

## Serialization and revival

```ts
import {EntropyCalculator} from 'harmonic-entropy';

const calc = new EntropyCalculator({N: 1000});
const serialized = JSON.stringify(calc);

const revived = JSON.parse(serialized, EntropyCalculator.reviver);
console.log(revived.ofCents(700));
```

## Notes and caveats

- Constructing `EntropyCalculator` performs precomputation immediately.
- Lower `res` (finer step) usually improves interpolation fidelity but costs more memory/time.
- `ofCents` rejects values outside `[minCents, maxCents]`.

## Background reading

- XenWiki article on harmonic entropy and consonance:
  <https://en.xen.wiki/w/Harmonic_entropy>
