<script setup>
// 引入组合式API
import { ref, onMounted } from 'vue'

// 引入threejs
import * as THREE from 'three'

// 引入轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

// 加载hdr环境贴图
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js'

// 引入GLTF加载器
// import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

// 用Draco库压缩的几何装载器。
// import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

// 画布节点
const sceneDom = ref(null)

// 创建场景
const scene = new THREE.Scene()

// 创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)
camera.position.set(0, 1.5, 6)
scene.add(camera)

// 创建渲染器
const renderer = new THREE.WebGLRenderer({
  antialias: true,
  preserveDrawingBuffer: true
})
renderer.setSize(window.innerWidth, window.innerHeight)

// 创建辅助坐标轴
const axesHelper = new THREE.AxesHelper(5)
scene.add(axesHelper)

// 侦听屏幕宽高改变,调整渲染器宽高和相机大小控制器
function handResize () {
  camera.aspect = (window.innerWidth / window.innerHeight)
  camera.updateProjectionMatrix ()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(() => {
  // 将场景添加到画布上
  sceneDom.value.appendChild( renderer.domElement )

  // 创建轨道控制器
  const controls = new OrbitControls( camera, sceneDom.value )
  controls.enableDamping = true
  controls.update()

  // 创建rgbe加载器
  const rgbeLoader = new RGBELoader()
  //资源较大，使用异步加载
  rgbeLoader.loadAsync('/assets/sky12.hdr').then(texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    scene.background = texture
    scene.environment = texture
  })

  // 渲染函数
  function animate() {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render( scene, camera )
  }

  // 侦听屏幕宽高改变,调整渲染器
  window.addEventListener('resize', handResize)

  animate()
})
</script>

<template>
  <div class="canvas" ref='sceneDom'></div>
</template>

<style scoped lang="less">
  .canvas {
    width: 100vw;
    height: 100vh;
  }
</style>
