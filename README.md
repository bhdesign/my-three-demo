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
| Lights | `HemisphereLight` for ambient fill, plus a `DirectionalLight` (shadows disabled) |
| Ground | 20×20 plane with a 20×20 `GridHelper` on top |
| Boxes | 100 static boxes in a 10×10 grid, 1.5 units apart, all the same purple (`#aa3bff`) |
| FPS counter | `Stats` panel in the top-left corner; click it to cycle FPS / ms per frame / MB memory |

### The 100 boxes

Each box is a **separate `Mesh` with its own `BoxGeometry` and its own `MeshStandardMaterial`**. Nothing is shared or instanced. This is deliberate: the scene is meant to show the cost of many individual meshes (100 draw calls per frame for the boxes). Shadows and animation are off and every box is the same colour, so the FPS counter reflects mainly the per-mesh overhead.

- Boxes are named `box-0` to `box-99`, so you can find one with `scene.getObjectByName('box-42')`.
- The boxes don't move, but the scene is still re-rendered every frame so the FPS counter measures the real draw cost.
- Although they share one colour, each box still has its own material instance.
- Change the grid size, gap and colour with the `GRID`, `SPACING` and `BOX_COLOR` constants.

### Lifecycle

- **On mount:** the scene is built, and a `ResizeObserver` keeps the camera and renderer matched to the container size.
- **On unmount:** the render loop stops, the observer, controls and FPS panel are released, every geometry and material is disposed, and the canvas is removed.
