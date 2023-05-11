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
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

// 用Draco库压缩的几何装载器。
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

// Reflector 镜面反射效果
import { Reflector } from 'three/addons/objects/Reflector.js'

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

// 实例化GLTF加载器（压缩、DRACO加载器（解压
const gltfLoader = new GLTFLoader()
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('/draco/gltf/')
dracoLoader.setDecoderConfig({ type: 'js' })
gltfLoader.setDRACOLoader( dracoLoader )
gltfLoader.load('/assets/robot.glb', gltf => {
  const robot = gltf.scene
  scene.add(robot)
})

// 添加平行光
const light1 = new THREE.DirectionalLight( 0xffffff, 1 )
light1.position.set(0, 10, 10)

const light2 = new THREE.DirectionalLight( 0xffffff, 1 )
light2.position.set(0, 10, -10)

const light3 = new THREE.DirectionalLight( 0xffffff, 1 )
light3.position.set(10, 10, 10)

scene.add(light1, light2, light3)



onMounted(() => {
  // 将场景添加到画布上
  sceneDom.value.appendChild( renderer.domElement )

  // 创建视频
  const video = document.createElement('video')
  video.src = '/assets/zp2.mp4'
  // 媒介文件完成播放后再次开始播放
  video.loop = true
  // 视频的音频输出为静音。
  video.muted = true
  video.play()

  // 创建光圈
  const videoTexture = new THREE.VideoTexture(video)
  const videoGeometry = new THREE.PlaneGeometry(16, 9)
  const videoMaterial = new THREE.MeshBasicMaterial({
    map: videoTexture,
    // 此材质是否透明
    transparent: true,
    // 定义将要渲染哪一面 - 正面，背面或两者
    side: THREE.DoubleSide,
    opacity: 1.0,
    // alpha贴图是一张灰度纹理，用于控制整个表面的不透明度。
    // （黑色：完全透明；白色：完全不透明）。 默认值为null
    alphaMap: videoTexture
  })
  const videoMesh = new THREE.Mesh(videoGeometry, videoMaterial)
  videoMesh.position.set(0, 0.2, 0)
  videoMesh.rotation.x = -Math.PI/2
  scene.add(videoMesh)

  // 创建镜面反射效果
  
  const reflectorPlane = new Reflector(videoGeometry, {
    textureWidth: window.innerWidth,
    textureHeight: window.innerHeight,
    color: 0x333333,
    recursion: 1
  })
  reflectorPlane.rotation.x = -Math.PI / 2
  scene.add( reflectorPlane )

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
