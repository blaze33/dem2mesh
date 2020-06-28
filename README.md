<div align="center">

  <h1><code>dem2mesh</code></h1>

  <strong>A wasm library to quickly process DEM heightmaps for <a href="https://threejs.org/">three.js</a>.</strong>

  <p>
    <a href="https://www.npmjs.com/package/dem2mesh"><img alt="npm version" src="https://img.shields.io/npm/v/dem2mesh"></a>
    <img alt="npm bundle size" src="https://img.shields.io/bundlephobia/minzip/dem2mesh?label=minified%20size">
  </p>

  <img alt="from heightmap to 3D terrain" src="https://raw.githubusercontent.com/blaze33/dem2mesh/master/docs/img/dem2mesh_banner.jpg">

</div>

## Overview

### Installation

```
npm install dem2mesh
```
alternatively:
```
yarn add dem2mesh
```

### Initialization

```js
let dem2mesh
import('dem2mesh').then(pkg => {
  dem2mesh = pkg
  dem2mesh.init()
}
```

### Usage

Currently works with 256x256px PNG images as input.
The PNG images come from the [terrain tile dataset hosted on Amazon](https://registry.opendata.aws/terrain-tiles/).

cf. [Get started with Mapzen Terrain Tiles](https://github.com/tilezen/joerd/blob/master/docs/use-service.md).

The current s3 url format is:

`https://s3.amazonaws.com/elevation-tiles-prod/terrarium/${z}/${x}/${y}.png`

Fetch one png tile to work with:
```js
let png
fetch(tileURL)
  .then(res => res.arrayBuffer())
  .then(arrayBuffer => png = new Uint8Array(arrayBuffer))
```

Once initialized the `dem2mesh` exposes two functions:
  - `dem2mesh.png2elevation`
  - `dem2mesh.png2mesh`

#### `dem2mesh.png2elevation(png)`

Returns a `Float32Array` of size `256*256` containing elevation data which can be used to set the positions of a `PlaneBufferGeometry` geometry.

```js
const heightmap = dem2mesh.png2elevation(png)
const geometry = new PlaneBufferGeometry(size, size, segments + 2, segments + 2)
const nPosition = Math.sqrt(geometry.attributes.position.count)
const nHeightmap = Math.sqrt(heightmap.length)
const ratio = nHeightmap / (nPosition)
let x, y
for (let i = nPosition; i < geometry.attributes.position.count - nPosition; i++) {
  if (
    i % (nPosition) === 0 ||
    i % (nPosition) === nPosition - 1
  ) continue
  x = Math.floor(i / (nPosition))
  y = i % (nPosition)
  geometry.attributes.position.setZ(
    i,
    heightmap[Math.round(Math.round(x * ratio) * nHeightmap + y * ratio)] * 0.075
  )
}
```

#### `dem2mesh.png2mesh(png, size, segments)`

Returns 3 arrays, position, index and uv wich can be used to create a three.js BufferGeometry.

```js
const [position, index, uv] = dem2mesh.png2mesh(png, size, segments)
const geometry = new BufferGeometry()
geometry.setAttribute(
  'position',
  new BufferAttribute(Float32Array.from(position), 3)
)
geometry.setIndex(
  new BufferAttribute(Uint16Array.from(index), 1)
)
geometry.setAttribute(
  'uv',
  new BufferAttribute(Float32Array.from(uv), 2)
)
geometry.computeVertexNormals()
```

## About

This package was built upon the `wasm-pack-template`.

[**📚 Read the template tutorial! 📚**][template-docs]

The `wasm-pack-template` is designed for compiling Rust libraries into WebAssembly and
publishing the resulting package to NPM.

Be sure to check out [other `wasm-pack` tutorials online][tutorials] for other
templates and usages of `wasm-pack`.

[tutorials]: https://rustwasm.github.io/docs/wasm-pack/tutorials/index.html
[template-docs]: https://rustwasm.github.io/docs/wasm-pack/tutorials/npm-browser-packages/index.html

## 🚴 Contribution

### 🐑 Clone this Template

```
git clone https://github.com/blaze33/dem2mesh.git
cd dem2mesh
```

### 🛠️ Build with `wasm-pack build`

```
wasm-pack build
```

### 🔬 Test in Headless Browsers with `wasm-pack test`

```
wasm-pack test --headless --firefox
```

## 🔋 Batteries Included

* [`wasm-bindgen`](https://github.com/rustwasm/wasm-bindgen) for communicating
  between WebAssembly and JavaScript.
* [`console_error_panic_hook`](https://github.com/rustwasm/console_error_panic_hook)
  for logging panic messages to the developer console.
* [`wee_alloc`](https://github.com/rustwasm/wee_alloc), an allocator optimized
  for small code size.
