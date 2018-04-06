# `import.meta` compiler extensions

This is a proposal for some community extensions to `import.meta` which makes
it easier for tooling that relies on static analysis of imports to better
integrate when it comes to dynamic imports.

## Problem

Unlike `import "url"` declarations `import(url)` expressions don't have a
static url declared as a string. It is perfectly valid to dynamically create
urls

**Example 1:**

```js
let puppiesUrl = "./images/puppies.jpg";
let kittiesUrl = "./images/kitties.jpg";
let animalUrl = Math.random() ? puppiesUrl : kittiesUrl;

import(animalUrl).then(img => ...);
```

**Example 2:**

```js
let animal = Math.random() ? "puppies" : "kitties";
let animalUrl = "./images/" + animal;

import(animalUrl).then(img => ...);
```

This is a good feature to have, but it does cause problems for bundlers, type
checkers, linters, and other tools that rely on static analysis to find module
dependencies.

For example, with the above examples it is difficult to reliably determine
which modules are going to be imported and there are even more situations where
it is impossible altogether.

We need a way in JavaScript to describe dependencies without immediately
importing them.

## `import.meta.url("./path")`

```js
let puppiesUrl = import.meta.url("./images/puppies.jpg");
let kittiesUrl = import.meta.url("./images/kitties.jpg");
let animalUrl = Math.random() ? puppiesUrl : kittiesUrl;

import(animalUrl).then(img => ...);
```

**Desugared:**

```js
let puppiesUrl = "./images/puppies.jpg";
let kittiesUrl = "./images/kitties.jpg";
let animalUrl = Math.random() ? puppiesUrl : kittiesUrl;

import(animalUrl).then(img => ...);
```

## `import.meta.glob("./glob/*")`

```js
let getImageUrl = import.meta.glob("./images/*");
let animal = Math.random() ? "puppies" : "kitties";
let animalUrl = getImageUrl(`./images/${animal}`);

import(animalUrl).then(img => ...);
```

**Desugared:**

```js
function __importMetaGlob(glob, regex) {
  return url => {
    if (regex.test(url)) {
      return url;
    } else {
      throw new Error(`${url} does not match ${glob}`);
    }
  };
}

let getImageUrl = __importMetaGlob("./images/*", /^\.\/images\/([^/]*)$/);
let animal = Math.random() ? "puppies" : "kitties";
let animalUrl = getImageUrl(`./images/${animal}`);

import(animalUrl).then(img => ...);
```

## Alternatives

### `() => import("./path")`

Today the easiest way to create static dependencies without immediately
importing them is to wrap them in a function.

```js
let importPuppies = () => import("./images/puppies.jpg");
let importKitties = () => import("./images/kitties.jpg");
let importAnimal = Math.random() > 0.5 ? importPuppies : importKitties;

importAnimal().then(img => ...);
```

This works okay for the `import.meta.url` case, but doesn't solve the
`import.meta.glob` case.
