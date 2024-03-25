# Testing Fundamentals

_"Effortless as the wind that whispers by, when upon my bike I ride and fly."_<!--more-->

<!-- mtoc-start -->

* [Intro](#intro)
* [Brief Intro to Vite](#brief-intro-to-vite)
  * [The big why](#the-big-why)
  * [Basic structure of a Vite project](#basic-structure-of-a-vite-project)
  * [Working with CSS modules](#working-with-css-modules)
  * [Vite with JS frontend framework](#vite-with-js-frontend-framework)
  * [Using Vite project template](#using-vite-project-template)
  * [Working with static Assets](#working-with-static-assets)
  * [`assetsInclude`](#assetsinclude)
  * [vite-imagetools](#vite-imagetools)
  * [`import.meta`](#importmeta)
  * [Conditional imports](#conditional-imports)
  * [vite-plugin-dynamic-import](#vite-plugin-dynamic-import)
  * [`import.meta.glob`](#importmetaglob)
  * [Environments and modes](#environments-and-modes)
  * [Create a component library](#create-a-component-library)
  * [Integrate `solid-js` with `vite`](#integrate-solid-js-with-vite)
* [Kinds of tests](#kinds-of-tests)
  * [unit tests](#unit-tests)
  * [integration tests (functional)](#integration-tests-functional)
  * [system tests (e2e)](#system-tests-e2e)
  * [acceptance tests: mimic real user behavior](#acceptance-tests-mimic-real-user-behavior)
* [Unit test with Vitest](#unit-test-with-vitest)
  * [Common commands with Vitest](#common-commands-with-vitest)
  * [`it` and `test`](#it-and-test)
  * [Equality](#equality)
  * [Throw Error?](#throw-error)
  * [Skip tests](#skip-tests)
  * [Compare tests btw commits: `vitest --changed`](#compare-tests-btw-commits-vitest---changed)
  * [Test suites: `describe`](#test-suites-describe)
  * [Test Asynchronous code](#test-asynchronous-code)
  * [Asymmetric matcher](#asymmetric-matcher)
  * [Parameterizing tests: `it.each` or `describe.each`](#parameterizing-tests-iteach-or-describeeach)
  * [Concurrent tests](#concurrent-tests)

<!-- mtoc-end -->

## Intro

First, why we write tests?

- **Laziness**: we write tests to make writting high-quality code easy
- **Good Dev. Env.**: set up your env so that **writing/running** the test is easier than the application
  - this reduces effort to get to the erroneous states
- **Tests Driving Dev.**: think of tests as the driving force to write high-quality code
  - when you have test-first mindset, you will end up with tests that are easy to reason about

## Brief Intro to Vite

Vite is a build tool and development server that is designed to make web development, particularly for modern JavaScript applications, faster and more efficient.

### The big why

- Vite has a fast server for development/testing. Like nodemon, using Vite also has Hot Module Replacement features.
- It allows the use of `import` and `export` without compiling JS/TS code first
- Provide a flexible configuration interface through the file `vite.config.js`
- Vite handles assets like imgs, fonts, and other static resources, allowing you to import them in the code directly
- Vite supports import of CSS modules
- Vite's functionality is extendable through plugins
- Vite analyzes codebase automatically to eliminate dead code paths, reducing bundle size when building the codebase

### Basic structure of a Vite project

```bash
project-root
├── vite.config.js # vite configuration file
├── index.html # has <script type="module" src="/src/index.js"></script>
├── public # static content
└── src
    ├── index.js
    └── components # ui components
    ... # other stuff
```

In `package.json` of a `vite`-enabled project, we usually prepare the following scripts for developing, testing, and building the app. If you're using `npm` or `bun`, you can run any one of the folowing scripts through `npm/bun run xxx`.

```json
  "scripts": {
    "start": "vite --config vite.config.js",
    "dev": "vite --config vite.config.js",
    "build": "tsc && vite build --config vite.config.js",
    "test": "vitest",
  },
```

For `bun` users, you can follow the instructions [here](https://bun.sh/guides/ecosystem/vite) to prepare the scripts below:

```json
"dev": "bunx --bun vite",
"build": "bunx --bun vite build --config vite.config.js",
"test": "vitest",
```

Notice that building the application will **result in** a folder called `dist` in your project folder, in which you will find compiled JS under the subfolder called `assets`.

### Working with CSS modules

If we give a CSS file a `*.module.css` extension, then we can access its fingerprinted classes.

```css
/*in style.modue.css*/
.count {
  font-size: 4em;
  color: rebeccapurple;
}
```

```javascript
/* in xxxx.js */
import styles from "style.module.css";
document.getElementById("count").classList.add(styles.count);
```

Vite can also handle `SCSS` and [PostCSS](https://postcss.org/).

### Vite with JS frontend framework

Check community support for Vite to work with various JS frontend framework [here](https://github.com/vitejs/awesome-vite?tab=readme-ov-file#plugins).

### Using Vite project template

You can quickly scaffold out a new Vite project by using `npm create vite@latest` or `bun create vite@latest`, which will start a cli interface to ask you project name(create a new folder with given name) and choose frontend framework(options are Vanilla, Vue, React, Preact, Solid,...)

A good template for using Vite with `typescript` can be found [here](https://github.com/stevekinney/template-typescript)

### Working with static Assets

By default, Vite serves static files from the `public` directory in the root of your project. Files in this directory are served as-is at the root level.

For example, if you have an image at `public/my-image.jpg`, it will be available at `http://localhost:3000/my-image.jpg.`

There are some key points to remember when keeping static content in `public` folder:

- Unlike files in src or other source code directories, changes to files in public don’t trigger a rebuild.
- Files in the public directory cannot be imported in your source code as modules.
- These files are not processed or optimized by Vite or any of its plugins.

**BUT YOU CAN KEEP STATIC CONTENT IN `src` FOLDER TO PROCESS IT**, see the section of [vite-imagetools](#vite-imagetools).

Let's say you have a `image.jpg` placed at `src/images`. Then you can create an image element in JS/TS like the following:

```javascript
import image from "./images/image.jpg";

const content = document.querySelector("#content");

export default function loadImage() {
  const imageElement = document.createElement("img");
  imageElement.src = image;
  content.appendChild(imageElement);
}
```

If you're facing type error when using the snippet above, you can create a `vite.d.ts` file at the root with the following content in it:

```typescript
/// <reference types="vite/client" />
```

### `assetsInclude`

By default, Vite treats certain types of files like images and fonts as assets, meaning they will be copied as-is into the final build directory without being bundled as JavaScript. However, you may have other custom file types that you want to be treated in the same way. That’s where assetsInclude comes in handy.

You can use this option to extend the default behavior by specifying a string,

```javascript
// in vite.config.js
import { defineConfig } from 'vite'
export default defineConfig{
  assetsInclude: ["*.pdf", "*.ico"],
};
// in another .js file
import pdfPath from "src/assets/some-document.pdf";
```

by using a RegExp,

```javascript
import { defineConfig } from 'vite'
export default defineConfig{
	assetsInclude: /^./some-special-dir/.*.special-extension$/
};
```

by array of patterns

```javascript
import { defineConfig } from 'vite'
export default defineConfig {
	assetsInclude: ['*.pdf', /^./some-special-dir/.*.special-extension$/]
};
```

### vite-imagetools

To install vite-imagetools, simply run `npm install -D vite-imagetools` or `bun run -d vite-imagestools`, and in your `vite.config.js` add the following line in `defineConfig`:

```javascript
plugins: [imagetools()];
```

You **MUST** move your images to `src` folder before using imagetools to process them. For example, the `import` command below converts a large JPG image to a 100x100 webpg image:

```javascript
// image is in the src folder
import image from "./large-img.jpg?h=100&w=100&format=webp";
```

### `import.meta`

- Variables prefixed with VITE\_ in your .env files are exposed to your code via `import.meta.env.VITE_XXX`
- `import.meta.url` provides the URL of the current module, which can be useful for dynamic imports or working with assets.

### Conditional imports

In Vite, when you use the `import()` function to dynamically import a module, it returns a Promise that resolves to the module object. This module object contains all the exported members of the imported module.

Here's an example:

```javascript
import("/path/to/module.js").then((module) => {
  // Use the imported module, assume it exports a function
  // called foo
  console.log(module.foo());
});
```

Vite offers more advanced features like the define option in `vite.config.js`, which lets you replace global variables at compile time:

```javascript
// vite.config.js
export default defineConfig{
  define: {
	__MY_CONDITION__: 'some value'
  }
};
// some other js/ts file
if (__MY_CONDITION__ === 'some value') {
	// Conditionally import something
}
```

### vite-plugin-dynamic-import

While Vite doesn’t offer built-in direct support for conditional imports in import statements, there’s a third-party plugin called [ vite-plugin-dynamic-import ](https://www.npmjs.com/package/vite-plugin-dynamic-import) that provides such a feature.

```javascript
//vite.config.js
import { defineConfig } from 'vite'
import dynamicImport from 'vite-plugin-dynamic-import';

export default defineConfig {
	plugins: [dynamicImport()]
};
// in other js/ts file
import(`./content/${variable}.js`);
```

### `import.meta.glob`

Vite includes support for importing multiple modules at the same time `import.meta.glob`. For example, you should see **something where the keys are the file names and the values are promises that are just import statements** by running

```javascript
console.log(import.meta.glob("./logos/**/*.svg"));
```

This means, we could do smt like:

```javascript
const content = document.querySelector("#content");

export default function logos() {
  const modules = import.meta.glob("./logos/**/*.svg");

  for (const m of Object.values(modules)) {
    m().then((svg) => {
      const img = document.createElement("img");
      img.width = 100;

      img.src = svg.default;
      content.appendChild(img);
    });
  }
}
```

### Environments and modes

Vite has the following built-in variables:

- MODE: Running mode (development, production, etc.)
- BASE_URL: Base URL of the app
- PROD: Boolean, indicating if in production
- DEV: Boolean, indicating if in development
- SSR: Boolean, indicating if in server-side rendering mode

Vite supports multiple .env files for different modes:

- `.env`: Default.
- `.env.local`: Local overrides for the default environment.
- `.env.[mode]`: For a specific mode (e.g., .env.production for production mode).
- `.env.[mode].local`: Local overrides for a specific mode.

In your application, you can access these variables through `import.meta.env.VARIABLE_NAME`.

Since environment variables are strings, you may want to declare their types if you’re using TypeScript. Create a `env.d.ts` file for type declarations:

```javascript
interface ImportMetaEnv {
	VITE_API_URL: string;
	// define other variables here
}
```

some security concerns:

- `.env.*.local` files are local-only and should not be checked into version control.
- No sensitive data should be prefixed with VITE\_ as it will be exposed to the client.

### Create a component library

We first give Vite an entry point for our library. For example, `src/components/index.ts`, in which we have two components:

```typescript
export * from "./button";
export * from "./input";
```

We then update `vite.config.js` to set up library generation:

```javascript
import { resolve } from "node:path";
import { defineConfig } from "vite";
// install by npm install -D
// or bun install -d
// the two plugins below are for
// generate index.d.ts in dist folder
// and resolve import css issues in ts
import dts from "vite-plugin-dts";
import { libInjectCss } from "vite-plugin-lib-inject-css";

// find current root directory path
const __dirname = new URL(".", import.meta.url).pathname;

export default defineConfig({
  plugins: [libInjectCss(), dts({ include: ["src/components"] })],
  build: {
    // not include publich folder
    copyPublicDir: false,
    // for injecting css
    cssCodeSplit: true,
    lib: {
      entry: resolve(__dirname, "src/components/index.ts"),
      name: "libraryName",
      // will crete libname.js and libname.umd.cjs
      fileName: "libname",
      formats: ["es", "umd"],
    },
  },
});
```

You can add your library to your `package.json` like the following:

```json
{
  "main": "./dist/libname.umd.cjs",
  "module": "./dist/libname.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/libname.js",
      "require": "./dist/libname.umd.cjs",
      "types": "./dist/index.d.ts"
    }
  }
}
```

### Integrate `solid-js` with `vite`

- [vite-plugin-solid](https://github.com/solidjs/vite-plugin-solid)
- [ts-vitest-template](https://github.com/solidjs/templates/tree/main/ts-vitest)

## Kinds of tests

### unit tests

- focusing on individual units like functions or classes (how things are done)
- fastest in speed
- highly isolated from other parts of your codebase
- easy to maintain, low cost to execute
- provides the lowest level of confidence for your code quality

### integration tests (functional)

- how units interact with a module or subsystem
- slightly slower than unit test
- less isolated than unit test
- more expensive than unit test
- harder to maintain but gives more confidence than unit test

### system tests (e2e)

- covering entire system as a whole
- hard to maintain, execute, and have minimal isolation
- gives system-level confidence

### acceptance tests: mimic real user behavior

- extra tool outside of your codebase might be required to run the tests

## Unit test with Vitest

[Vitest](https://vitest.dev/), due to its compatibility with Vite, is commonly used for unit tests. This setion, we give some simple examples of using `it`, `test`, and `expect` from Vitest.

To start with Vitest, you need to have a `vite.config.ts`(see [vite intro](#brief-intro-to-vite)) a `vitest.config.ts`.

### Common commands with Vitest

Assume you're using `bun`(`bun run`==`npx`).

- `bun run vitest`: run all your tests and then watch for changes
- `bun run vitest --ui`: above + open vitest UI
- `bun run vitest --dom`: mock browser APIs using jsdom
- `bun run vitest --browser`: run your tests in browser, require `@vitest/browser`

### `it` and `test`

`it` and `test` are synonyms. Below is a simple test on the function `add`:

```typescript
import { expect, it } from "vitest";

const add = (a: number, b: number) => a + b;

it("should work as expected", () => {
  expect(add(2, 4)).toBe(6);
});
```

The test above passes because 2+4 is indeed 6. However, things get more complicated if we are testing asynchronous code. For example, the test below also passes, even though it should not:

```typescript
it('works with "test" as well', () => {
  setTimeout(() => {
    expect("This should fail.").toBe("Totally not the same.");
  }, 1000);
});
```

You can workaround this by making sure `expect` command is called,

```typescript
it('works with "test" as well', () => {
  expect.hasAssertions();
  setTimeout(() => {
    expect("This should fail.").toBe("Totally not the same.");
  }, 1000);
});
```

### Equality

There is many types of equalities in `vitest`:

- `toBe`: strict equality
- `toEqual`: asserts if actual value is equal to received one or has the same structure
  > ` expect([1, [2, 3]]).toEqual([1, [2, 3]]);`
- `toBeCloseTo`: use this to compare floating-point numbers
- `toBeInstanceOf`: check if a variable is an instance of certain class
  > `const s=new Stock(); expect(stocks).toBeInstanceOf(Stocks);`
- `toBeUndefined`: asserts that the value is `undefined`
- `toBeTruthy`: asserts that the value is true when converted to boolean
- `toCOntain`: asserts if the actual value is in an array.

### Throw Error?

You can use `toThrow` or `toThrowError` to check if a function throws error:

```typescript
test("will throw if you provide an empty string", () => {
  const fn = () => {
    new Person("");
    throw new Error("empty string");
  };

  expect.hasAssertions();
  expect(() => fn()).toThrowError();
  // Verify that function above throws.
  expect(() => fn()).toThrowError("empty string");
  // verify the error msg is correct
});
```

### Skip tests

Sometimes, we don't want all of our tests to run.

- `it.skip`: Skip this test for now.
- `it.only`: Only run this and any other test that uses .only.
- `it.todo`: Note to self—I still need to write this test.
- `it.fails`: Yea, I know this one fails. Don't blow up the rest of the test run, please.

### Compare tests btw commits: `vitest --changed`

Runs tests related to files that changed. Out of the box, this will be against any uncommitted files. But, you can also do cool stuff like `--changed HEAD~1` or give it a branch name to compare to or a particular commit.

### Test suites: `describe`

You can group a set of tests into a suite using `describe`. If you don't use `describe`, all of the tests in a given file as grouped in a suite automatically.

```typescript
import { describe, it, expect } from "vitest";
import { add, subtract } from "./math";

// run all the tests in this suite concurrently
describe.concurrent("add", () => {
  it("should add two numbers correctly", () => {
    expect(add(2, 2)).toBe(4);
  });

  it("should not add two numbers incorrectly", () => {
    expect(add(2, 2)).not.toBe(5);
  });
});

// we skip the following suite
describe.skip("subtract", () => {
  it("should subtract the subtrahend from the minuend", () => {
    expect(subtract(4, 2)).toBe(2);
  });

  it("should not subtract two numbers incorrectly", () => {
    expect(subtract(4, 2)).not.toBe(1);
  });
});
```

### Test Asynchronous code

**Method 1: use `async`/`await`**

Consider the following:

```typescript
const addAsync = (a: number, b: number) => Promise.resolve(a + b);

test("passes if use an `async/await`", async () => {
  const result = await addAsync(2, 3);
  expect(result).toBe(5);
});
```

**Method 2: use `expect.resolve/rejects`**

```typescript
it("passes if we expect it to resolve", () => {
  const result = addAsync(2, 3);
  expect(result).resolves.toBe(5);
});

it("passes if we expect it to reject to particular value", () => {
  const result = onlyEvenNumbers(5);
  expect(result).rejects.toBe(5);
});
```

### Asymmetric matcher

Asymmetric matchers allow you to **ONLY** focus on the things you care about. Consider the following type and function:

```typescript
import { v4 as id } from "uuid";
import { expect, it } from "vitest";

type ComputerScientist = {
  id: string;
  firstName: string;
  lastName: string;
  isCool?: boolean;
};

const createComputerScientist = (
  firstName: string,
  lastName: string,
): ComputerScientist => ({ id: "cs-" + id(), firstName, lastName });

const addToCoolKidsClub = (p: ComputerScientist, club: unknown[]) => {
  club.push({ ...p, isCool: true });
};
```

Now, if we add multiple computer scientists to an array caleed `people`,

```typescript
addToCoolKidsClub(createComputerScientist("Grace", "Hopper"), people);
addToCoolKidsClub(createComputerScientist("Ada", "Lovelace"), people);
addToCoolKidsClub(createComputerScientist("Annie", "Easley"), people);
addToCoolKidsClub(createComputerScientist("Dorothy", "Vaughn"), people);
```

Let's say we just cared if they're cool and they they have a first and last name that are strings, we can use Vitest to do the following:

```typescript
for (const person of people) {
  expect(person).toEqual({
    id: expect.stringMatching(/^cs-/),
    firstName: expect.any(String),
    lastName: expect.any(String),
    isCool: true,
  });
}
```

Alternatively, if we're just looking for one property, we can do the following:

```typescript
for (const person of people) {
  expect(person).toEqual(
    expect.objectContaining({
      isCool: expect.any(Boolean),
    }),
  );
}
```

### Parameterizing tests: `it.each` or `describe.each`

describe.each and it.each (or, test.each) allow us to use an array or table to automatically generate tests for ourselves.

```typescript
export class Polygon {
  sides: number;
  length: number;

  constructor(sides: number, length: number) {
    if (sides < 3) throw new Error("Polygons must have three or more sides.");
    this.sides = sides;
    this.length = length;
  }

  get type(): PolygonType | undefined {
    if (this.sides === 3) return "triangle";
    if (this.sides === 4) return "quadrilateral";
    if (this.sides === 5) return "pentagon";
    if (this.sides === 6) return "hexagon";
    if (this.sides === 7) return "heptagon";
    if (this.sides === 8) return "octagon";
    if (this.sides === 9) return "nonagon";
    if (this.sides === 10) return "decagon";
  }
}
```

And a test for `get type()` could simply be:

```typescript
it.each([
  [3, 'triangle'],
  [4, 'quadrilateral'],
  [5, 'pentagon'],
  [6, 'hexagon'],
  [7, 'heptagon'],
  [8, 'octagon'],
  [9, 'nonagon'],
  [10, 'decagon'],
])('a polygon with %i sides should be considered a %s', (sides, type) => {
  const polygon = new Polygon(sides, 10);
  expect(polygon.type).toBe(type);
});
});
```

### Concurrent tests

In order to parallelize tests, you have to use test context. `it` and `test` take a function as a second argument, which receives the test context as a argument. The text context has two main properties:

- `meta`: some metadata about the test itself
- `expect`: a copy of the Expect API bound to the current test

They can be used as the following:

```typescript
it("should work", (ctx) => {
  expect(ctx.meta.name).toBe("should work");
});

it("should really work", ({ meta }) => {
  expect(meta.name).toBe("should really work");
});

it("should have version of `expect` bound to the current test", (ctx) => {
  ctx.expect(ctx.expect).not.toBe(expect);
});
```

You can even extend the context by defining extra `interface`:

```typescript 
interface LocalTestContext {
  foo: string;
}

beforeEach<LocalTestContext>(async (context) => {
  // typeof context is 'TestContext & LocalTestContext'
  context.foo = 'bar';

  it<LocalTestContext>('should work', ({ foo }) => {
    // typeof foo is 'string'
    console.log(foo); // 'bar'
  });
});
```

To run tests in a suite in parallel, you have to follow the two rules below:

1. You must use the verison expect bound to the test via the context argument passed to each test function (e.g. `context.expect`).
2. You must annotate either the individual tests that you want to run concurrently or the entire suite.

For example, the tests below will run in parallel: 

```typescript 
describe.concurrent('sleep', () => {
  it('should sleep for 500ms', async ({ expect }) => {
    await sleep(500);
    expect(true).toBe(true);
  });

  it('should sleep for 750ms', async ({ expect }) => {
    await sleep(750);
    expect(true).toBe(true);
  });

  it('should sleep for 1000ms', async ({ expect }) => {
    await sleep(1000);
    expect(true).toBe(true);
  });

  it('should sleep for 1500ms', async ({ expect }) => {
    await sleep(1500);
    expect(true).toBe(true);
  });
});
```
