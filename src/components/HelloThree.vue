// ----------hello threejs
<template>
<div ref="root"></div>
</template>

<script>
import * as THREE from 'three'
import {OrbitControls} from 'three/addons/controls/OrbitControls.js'
let scene = null;
let camera = null;
let renderer = null;
let controls = null;
export default {
  name: 'HelloThree',
  data() {
    return {
        cube: "",
    }
  },
  mounted() {
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000);
    renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth,window.innerHeight);
    this.$refs.root.appendChild(renderer.domElement);

    this.init();
    this.animate();
  },
  methods: {
    // 创建立方体
    init(){
        const geometry = new THREE.BoxGeometry(1,1,1);
        const material = new THREE.MeshBasicMaterial({
            color: '#0f0'
        })
        this.cube = new THREE.Mesh(geometry, material);
        scene.add(this.cube);

        camera.position.z = 5;
        // Orbit controls（轨道控制器）可以使得相机围绕目标进行轨道运动。
        controls = new OrbitControls(camera,renderer.domElement)
    },


    // 渲染场景
    animate(){
        requestAnimationFrame(this.animate);
        this.cube.rotation.x += 0.01;
        this.cube.rotation.y += 0.01;
        this.cube.rotation.z += 0.01;

        renderer.render(scene,camera)
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
