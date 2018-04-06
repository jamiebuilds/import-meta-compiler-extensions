# `import.meta` compiler extensions

This is a proposal for some community extensions to `import.meta` which makes
it easier for tooling that relies on static analysis of imports to better
integrate when it comes to dynamic imports.

## `import.meta.url("./path")`

```js
let puppiesUrl = import.meta.url("./images/puppies.jpg");
let kittiesUrl = import.meta.url("./images/kitties.jpg");
let animalUrl = Math.random() ? puppiesUrl : kittiesUrl;

import(animalUrl).then(img => ...);
```

## `import.meta.glob("./glob/*")`

```js
let getImageUrl = import.meta.glob("./images/*");
let animal = Math.random() ? "puppies" : "kitties";
let animalUrl = getImageUrl("./images/" + animal);

import(animalUrl).then(img => ...);
```
