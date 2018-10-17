---
name: react-cache
menu: APIs
---

# `react-cache`

`react-cache` is a reference React Suspense cache implementation maintained by the React team. 
It was previously known as `simple-cache-provider`. 
[The entire codebase](https://github.com/facebook/react/blob/master/packages/react-cache/src/ReactCache.js) is only about 400 lines of Flow-typed code.

## Important Concepts

- **reading**
- **invalidating**
- **preloading**

## `createCache` Example

*Current API: `reactCache.cache`*

`createCache` creates a cache that works with `createResource`. 
You can create as many as you like if you are worried about key clashes, but most apps only need a singleton.

```js
// cache.js
import { createCache } from "react-cache";

export let cache;
function initCache() {
  cache = createCache(initCache);
}
initCache();
```

## `createResource` and `resource.read(key)` Example 

*Current API: `reactCache.createResource`*

`createResource` creates a `resource` out of a callback returning a promise.

The primary method of a `resource` is `read`. A mental model of `resource.read(key)`:

- The resource checks against a key and a `cache`. 
- If cache miss:
  - It calls the callback to obtain the data fetch Promise
  - Throws the promise
  - When the promise resolves, stores the response in `cache` against the key
  - Start over
- If cache read succeeds:
  - Synchronous read of the cached value into the render function

```js
const FooResource = createResource(id => fetch(`/foo/${id}`)); // return a promise, or async/await

// inside render...
  const fooResponse = FooResource.read(id); // suspends if not in cache; renders if in cache
  return <div> {fooResponse} </div>
```

Alternative Example with Image Loading:

```js
const ImageResource = createResource(
  src =>
    new Promise(resolve => {
      const img = new Image();
      img.onload = () => resolve(src);
      img.src = src;
    })
);

// suspending <Img /> component
const Img = ({ src, alt, ...rest }) => (
  <img src={ImageResource.read(src)} alt={alt} {...rest} />
);
```

## `resource.preload(key)` 

`resource.preload(key)` is like `resource.read(key)` except it does not throw and suspend rendering.
Thus it has the effect of "warming" the cache in the background.


## `resource.invalidate()` 

`resource.invalidate()` clears the cache. In future we should have fine-grained key-based invalidation.

--- 

> Next: [hidden](/apis/hidden)

--- 

**Recommended Sources for further info:**

- none, see the Suspense docs.