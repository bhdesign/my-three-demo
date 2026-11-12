<script setup lang="ts">
import { onBeforeUnmount, onMounted, useTemplateRef } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const container = useTemplateRef<HTMLDivElement>('container')

let renderer: THREE.WebGLRenderer
let controls: OrbitControls
let resizeObserver: ResizeObserver
const disposables: { dispose(): void }[] = []

onMounted(() => {
  const el = container.value!
  const { clientWidth: width, clientHeight: height } = el

  // Scene
  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x16171d)

  // Camera
  const camera = new THREE.PerspectiveCamera(50, width / height, 0.1, 100)
  camera.position.set(12, 10, 14)

  // Renderer
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setSize(width, height)
  renderer.shadowMap.enabled = true
  el.appendChild(renderer.domElement)

  // Controls
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.target.set(0, 0.5, 0)

  // Lights
  scene.add(new THREE.HemisphereLight(0xffffff, 0x444444, 1.5))
  const dirLight = new THREE.DirectionalLight(0xffffff, 2.5)
  dirLight.position.set(5, 8, 5)
  dirLight.castShadow = true
  dirLight.shadow.mapSize.set(2048, 2048)
  Object.assign(dirLight.shadow.camera, { left: -10, right: 10, top: 10, bottom: -10 })
  scene.add(dirLight)

  // Ground
  const groundGeo = new THREE.PlaneGeometry(20, 20)
  const groundMat = new THREE.MeshStandardMaterial({ color: 0x2e303a })
  const ground = new THREE.Mesh(groundGeo, groundMat)
  ground.rotation.x = -Math.PI / 2
  ground.receiveShadow = true
  scene.add(ground)

  const grid = new THREE.GridHelper(20, 20, 0x555555, 0x333333)
  grid.position.y = 0.001
  scene.add(grid)

  // Boxes: 10x10 grid, each one a separate Mesh with its own geometry and
  // material (intentionally unoptimized: 100 draw calls, 100 shadow draws)
  const GRID = 10
  const SPACING = 1.5
  const COUNT = GRID * GRID
  const offset = ((GRID - 1) * SPACING) / 2
  const boxes: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>[] = []
  for (let i = 0; i < COUNT; i++) {
    const geo = new THREE.BoxGeometry(1, 1, 1)
    const mat = new THREE.MeshStandardMaterial({
      color: new THREE.Color().setHSL(0.75 + (i / COUNT) * 0.25, 0.8, 0.55),
    })
    const box = new THREE.Mesh(geo, mat)
    box.name = `box-${i}`
    box.position.set((i % GRID) * SPACING - offset, 0.75, Math.floor(i / GRID) * SPACING - offset)
    box.castShadow = true
    box.receiveShadow = true
    scene.add(box)
    boxes.push(box)
    disposables.push(geo, mat)
  }

  disposables.push(groundGeo, groundMat, grid.geometry, grid.material as THREE.Material)

  // Resize
  resizeObserver = new ResizeObserver(() => {
    const w = el.clientWidth
    const h = el.clientHeight
    camera.aspect = w / h
    camera.updateProjectionMatrix()
    renderer.setSize(w, h)
  })
  resizeObserver.observe(el)

  // Render loop
  const timer = new THREE.Timer()
  renderer.setAnimationLoop((time) => {
    timer.update(time)
    const t = timer.getElapsed()
    boxes.forEach((box, i) => {
      box.position.y = 0.75 + Math.sin(t * 2 + i * 0.3) * 0.25
      box.rotation.y = t * 0.8 + i * 0.1
    })
    controls.update()
    renderer.render(scene, camera)
  })
})

onBeforeUnmount(() => {
  renderer?.setAnimationLoop(null)
  resizeObserver?.disconnect()
  controls?.dispose()
  disposables.forEach((d) => d.dispose())
  renderer?.dispose()
  renderer?.domElement.remove()
})
</script>

<template>
  <div ref="container" class="three-container"></div>
</template>

<style scoped>
.three-container {
  width: 100%;
  height: 100%;
}
</style>
