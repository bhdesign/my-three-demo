# demo-threejs

A Three.js scene rendered inside a Vue 3 + TypeScript + Vite app.

## Getting started

```sh
yarn install
yarn dev      # start the dev server
yarn build    # type-check and build for production
yarn preview  # serve the production build
```

## The scene

Everything lives in [`src/components/ThreeScene.vue`](src/components/ThreeScene.vue). The canvas fills the whole window.

| Element | Details |
| --- | --- |
| Camera | `PerspectiveCamera` (50° FOV) at `(12, 10, 14)`, looking at `(0, 0.5, 0)` |
| Controls | `OrbitControls` with damping: drag to orbit, scroll to zoom, right-drag to pan |
| Lights | `HemisphereLight` for ambient fill, plus a `DirectionalLight` that casts shadows (2048×2048 shadow map covering the grid) |
| Ground | 20×20 plane that receives shadows, with a 20×20 `GridHelper` on top |
| Boxes | 100 boxes in a 10×10 grid, 1.5 units apart, coloured purple to magenta across the grid |

### The 100 boxes

Each box is a **separate `Mesh` with its own `BoxGeometry` and its own `MeshStandardMaterial`**. Nothing is shared or instanced. This is deliberate: the scene is meant to show the cost of many individual meshes (about 200 draw calls per frame, counting the shadow pass).

- Boxes are named `box-0` to `box-99`, so you can find one with `scene.getObjectByName('box-42')`.
- Every frame, each box spins around its Y axis and bobs up and down. A per-box phase offset turns this into a wave across the grid.
- Change the grid size and gap with the `GRID` and `SPACING` constants.

### Lifecycle

- **On mount:** the scene is built, and a `ResizeObserver` keeps the camera and renderer matched to the container size.
- **On unmount:** the render loop stops, the observer and controls are released, every geometry and material is disposed, and the canvas is removed.
