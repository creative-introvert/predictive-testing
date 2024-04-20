# prediction-testing

`prediction-testing` is a test runner, simlar to [Jest](https://jestjs.io/), [vitest](https://vitest.dev/) or [ava](https://github.com/avajs/ava), that focuses on testing predictive functions (e.g. search, auto-complete, or ML).

## When you SHOULD use `prediction-testing`

- you have tons of test cases, and your tests purely compare inputs and outputs of the function under test
- you are testing a statistical model, or a predictive function, where 100% successful test results are impossible or impractical
- you are testing a (flaky) legacy system

## When you SHOULD NOT use `prediction-testing`

- when your tests are few, and predominantely example-based; use the conventional test runners instead (jest, ava, vitest)
- when you require your testing framework to do all sorts of magic (auto-mocking, spies, etc)

## Before Getting Started

Before looking at code examples, some notes on the design philosohpy:

1. Unlike tools like jest or vitest, prediction-testing's CLI does not provide a
   runtime, but is imported as a library. Check the ["Why No Runtime?" section](#why-no-runtime)
   on the reasoning.
2. Though only required minimally, the library depends on using
   [effect](https://effect.website/) (the missing standard library for
   TypeScript). At minimum, your function under test has to return an [Effect](https://effect.website/docs/guides/essentials/the-effect-type).
   If your function doesn't already do so, checkout the [Usage](#usage) below, to see how to trivially convert it.

## Usage

With your package manager of choice, install the following packages:

```bash
@creative-introvert/prediction-testing
@creative-introvert/prediction-testing-cli
```

### With CLI

```ts
import * as CLI from '@creative-introvert/prediction-testing-cli';
import {Effect} from 'effect';

const myFunction = (input: number) => Promise.resolve(input * 2);

const opts = {
    testCases: [
        {input: 0, expected: 0},
        {input: 1, expected: 2},
        {input: 2, expected: 3},
        {input: 3, expected: 4},
        {input: 4, expected: 5},
    ],
    // Convert myFunction to Effect-returning.
    program: (input: number) => Effect.promise(() => myFunction(input)),
};

void CLI.run(opts);
```

Checkout `workspace/examples/src/with-cli` for more examples.

```bash
pnpx tsx <file-path>
# e.g.
pnpx tsx ./workspace/examples/src/with-cli/simple.ts
```

### Without CLI

For full control over execution, and if you're not afraid of using [effect](https://effect.website/)
you may simply import the runner functions individually.

```ts
import * as PT from '@creative-introvert/prediction-testing';
import {Console, Duration, Effect, Stream} from 'effect';

type Input = number;
type Result = number;

const program = (input: Input) =>
    Effect.promise(() => Promise.resolve(input * 2)).pipe(
        Effect.delay(Duration.millis(500)),
    );

const testCases: PT.TestCase<Input, Result>[] = [
    {input: 0, expected: 0},
    {input: 1, expected: 1},
    {input: 2, expected: 2},
    {input: 3, expected: 3},
    {input: 4, expected: 4},
];

await PT.testAll({
    testCases,
    program,
}).pipe(
    PT.runFoldEffect,
    Effect.tap(testRun => Console.log(PT.Show.summary({testRun}))),
    Effect.runPromise,
);
```

Checkout `workspace/examples/src/as-library` for more examples.

```bash
pnpx tsx <file-path>
# e.g.
pnpx tsx ./workspace/examples/src/as-library/simple.ts
```

## Why No Runtime?

Tools like jest and vitest provide a dedicated CLI, which the user can run,
separately from other entrypoints into their app. For example, jest provides
the `jest` binary, which, magically, finds all the test files, and executes
them, reporting on the results.

Unfortunately, this comes with a lot of complexity:

1. Unless the user provides their tests as ES5 JavaScript (not a thing these
   days), jest has to figure out how transpile/compile the source. Given the
   level of, let's call it, variation in JavaScript land (Typescript, and its
   many different configurations, custom runtimes like svelte, ESM vs CommonJS,
   etc), this is no easy feat. The amount of code required for enabling this easily
   overtrumps the actual test runner.
2. As `jest` (etc) essentially behave as a _framework_ (jest calls _your_ code)
   as opposed to a _library_ (you call jest), customization requires additional
   complexity in the framework, which now has to provide various entrypoints
   into its execution.

Alternatively, the library could hook into an existing test runner (I've seen
that vite provides some programatic context), but I have not yet looked deeper
into that. Though saving me from solving this problem myself, this would likely
come with its own limitations.

Providing `prediction-testing` as a pure library is not as satisfying and
convenient as a dedicated binary, but is both simpler, and more easily
customizable.

## TODO

- sqlite backend
- Basic performance measurements.
