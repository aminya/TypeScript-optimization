# TypeScript-optimization
Tests and benchmarks different codes in TypeScript for different JavaScript versions (ES5 vs ES6 and above).

Benchmarks are done inside Atom (using script package) and Webstorm.

### Traditional `for` vs `for-of` vs `for-in`- Looping ovr Arrays

- ES6 and above: traditional `for` is **faster** than `for-of`, which is faster than `for-in`
- ES5 and lower:Traditional `for` is **similar to** `for-of`, and both are faster than `for-in`.

This is true for array of number, string, etc, and for any array size (10, 100, 10000 are tested).

If you notice, you see by targeting ES5 the TypeScript compiler converts `for-of` to the `traditional-for`, and that makes it faster than the original `for-of`!! Actually, by setting `"downlevelIteration": true
`, you can make `for-of` slow in ES5 too!!!  To fix this issue you can use `npm run build` which uses `@babel/plugin-transform-for-of` to convert `for-of` to `traditional-for` ("loose" is faster than "assumeArray").

```typescript
// Traditional
  let sum = 0
  for (let i = 0, l = arr.length; i < l; ++i) {
    sum += arr[i]
  }

// for - of
  let sum = 0
  for (const a of arr) {
    sum += a
  }

// for - in
  let sum = 0
  for (const i in arr) {
    sum += arr[i]
  }
```
See the ./src for full explanation.

<details>
<summary>Benchmark-Result</summary>

    -------------------    
    array size of 10

    ES6 and ES2020:
    
    number array
    for_traditional x 92,928,200 ops/sec Â±2.29% (82 runs sampled)
    for_of x 19,458,768 ops/sec Â±1.14% (95 runs sampled)
    for_in x 1,791,886 ops/sec Â±1.47% (90 runs sampled)
    Fastest is for_traditional
    
    string array
    for_traditional_str x 61,545,102 ops/sec Â±0.84% (91 runs sampled)
    for_of_str x 35,545,222 ops/sec Â±1.28% (94 runs sampled)
    for_in_str x 1,843,325 ops/sec Â±1.65% (90 runs sampled)
    Fastest is for_traditional_str
    
    
    -------------------    
    array size of 100

    ES6 and ES2020:
    
    number array
    for_traditional x 9,637,735 ops/sec Â±0.28% (93 runs sampled)
    for_of x 2,318,205 ops/sec Â±0.49% (95 runs sampled)
    for_in x 247,436 ops/sec Â±2.10% (92 runs sampled)
    Fastest is for_traditional
    
    string array
    for_traditional_str x 7,374,552 ops/sec Â±0.51% (89 runs sampled)
    for_of_str x 4,132,730 ops/sec Â±0.58% (89 runs sampled)
    for_in_str x 256,286 ops/sec Â±2.32% (87 runs sampled)
    Fastest is for_traditional_str
    
    -------------------    
    array size of 10000

    ES6 and ES2020:

    number array
    for_traditional x 82,093 ops/sec Â±0.72% (93 runs sampled)
    for_of x 25,968 ops/sec Â±5.27% (76 runs sampled)
    for_in x 2,147 ops/sec Â±3.52% (86 runs sampled)
    Fastest is for_traditional
    
    for_traditional_str x 64,355 ops/sec Â±0.32% (90 runs sampled)
    for_of_str x 46,471 ops/sec Â±0.48% (92 runs sampled)
    for_in_str x 2,538 ops/sec Â±1.27% (88 runs sampled)
    Fastest is for_traditional_str

    ES5:

    number array
    for_traditional x 83,733 ops/sec Â±0.22% (92 runs sampled)
    for_of x 83,851 ops/sec Â±0.16% (95 runs sampled)
    for_in x 2,289 ops/sec Â±0.57% (94 runs sampled)
    Fastest is for_of,for_traditional
    
    string array
    for_traditional_str x 65,135 ops/sec Â±0.57% (89 runs sampled)
    for_of_str x 66,133 ops/sec Â±0.21% (93 runs sampled)
    for_in_str x 2,295 ops/sec Â±5.61% (83 runs sampled)
    Fastest is for_of_str,for_traditional_str
    
</details>

### Traditional `for` optimization

- in all versions: First three the codes are about the same, but full array lookup in the `for-head` is very slow.

Defining `arr` as `const` or `let` doesn't affect the speed.

```typescript
// for-traditional
  let sum = 0
  for (let i = 0, l = arr.length; i < l; ++i) {
    sum += arr[i]
  }
// for-traditional-const
  let sum = 0
  const l = arr.length
  for (let i = 0; i < l; ++i) {
    sum += arr[i]
  }

// for-traditional-length-lookup
  let sum = 0
  for (let i = 0; i < arr.length; ++i) {
    sum += arr[i]
  }

// for-traditional-array-lookup
// to only measure its effect on calling inside the for-head
let sum = 0;
for (let i = 0; i < arr_return().length; ++i) {
    sum += arr[i];
}
```

<details>
<summary>Benchmark-Result</summary>

    const arr

    ES2020:

    for-traditional x 111,107 ops/sec Â±0.38% (97 runs sampled)
    for-traditional-const x 111,392 ops/sec Â±0.19% (98 runs sampled)
    for-traditional-lookup x 111,242 ops/sec Â±0.22% (95 runs sampled)
    for-traditional-full-lookup x 1.77 ops/sec ±1.09% (9 runs sampled)
    Fastest is for-traditional,for-traditional-const,for-traditional-length-lookup

    ES 6:

    for-traditional x 111,197 ops/sec Â±0.18% (95 runs sampled)
    for-traditional-const x 111,209 ops/sec Â±0.18% (96 runs sampled)
    for-traditional-lookup x 111,111 ops/sec Â±0.13% (96 runs sampled)
    for-traditional-full-lookup x 1.78 ops/sec ±0.77% (9 runs sampled)
    Fastest is for-traditional,for-traditional-const,for-traditional-length-lookup

    ES5:

    for-traditional x 109,984 ops/sec ±0.67% (95 runs sampled)
    for-traditional-const x 110,267 ops/sec ±0.80% (91 runs sampled)
    for-traditional-length-lookup x 109,373 ops/sec ±0.74% (94 runs sampled)
    for-traditional-full-lookup x 1.68 ops/sec ±3.05% (9 runs sampled)
    Fastest is for-traditional

    let arr:

     ES2020:

    for-traditional x 111,310 ops/sec Â±0.23% (93 runs sampled)
    for-traditional-const x 111,201 ops/sec Â±0.36% (96 runs sampled)
    for-traditional-lookup x 111,430 ops/sec Â±0.20% (99 runs sampled)
    Fastest is for-traditional-length-lookup,for-traditional,for-traditional-const

    ES5:

    for-traditional x 110,594 ops/sec ±0.53% (94 runs sampled)
    for-traditional-const x 111,455 ops/sec ±0.14% (97 runs sampled)
    for-traditional-lookup x 111,463 ops/sec ±0.15% (96 runs sampled)
    Fastest is for-traditional-const,for-traditional-length-lookup

</details>


### `for-of` optimization

- in all versions: full array look-up in the `for-head` is much slower.

```typescript
// for-of
  let sum = 0
  for (const a of arr) {
    sum += a
  }
// for-of-full-array-lookup
  let sum = 0
  for (const a of arr_return()) {
    sum += a
  }
```

<details>
<summary>Benchmark-Result</summary>

    ES2020:

    for-of x 83,144 ops/sec ±0.52% (93 runs sampled)
    for-of-full-lookup x 13,930 ops/sec ±0.62% (95 runs sampled)
    Fastest is for-of

    ES 6:

    for-of x 83,036 ops/sec ±0.43% (95 runs sampled)
    for-of-full-lookup x 13,779 ops/sec ±0.90% (96 runs sampled)
    Fastest is for-of

    ES5:

    for-of x 110,799 ops/sec ±0.15% (96 runs sampled)
    for-of-full-lookup x 15,122 ops/sec ±0.74% (95 runs sampled)
    Fastest is for-of

</details>
