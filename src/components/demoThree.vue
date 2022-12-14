//----自己练着玩的demo
<template>
    <div>
        <div class="title">DEMO</div>
        <div class="container" ref="container"></div>
    </div>
  
</template>

<script>
import * as THREE from 'three'
import {OrbitControls} from 'three/addons/controls/OrbitControls.js'
import threeDHeatmapVue from './threeDHeatmap.vue'

let camera,renderer,scene,
    controls,
    container

export default {
    name:'demoThree',
    data() {
        return {
            cube:"",
            plane:"",
            sphere:"",
        }
    },
    mounted(){
        container = this.$refs.container;
        this.init()
    },
    methods: {
        init(){
            this.initScene();
            this.initCameraAndRenderer();
            this.addMesh();
            this.addLight();
            this.render();
        },
        initScene(){
            scene = new THREE.Scene()
            scene.rotateX(-0.7*Math.PI/2)
            scene.background = new THREE.Color('#999')
            // scene.rotateZ(-0.2*Math.PI/2)

        },
        initCameraAndRenderer(){
            // camera
            camera = new THREE.PerspectiveCamera(60,1,1,1000);
            camera.position.z = 3;

            // renderer
            renderer = new THREE.WebGLRenderer()
            renderer.setSize(container.clientWidth,container.clientHeight)
            renderer.shadowMap.enabled = true;//开启阴影效果

            // controls
            controls = new OrbitControls(camera,renderer.domElement)
            controls.addEventListener('change', this.render);
        },
        addMesh(){
            let texture = new THREE.TextureLoader().load(require('./../assets/earth.png'))

            let boxGeometry = new THREE.BoxGeometry(1,1,1);
            let material = new THREE.MeshBasicMaterial({
                color: '#0f0',
            })
            this.cube = new THREE.Mesh(boxGeometry, material)
            this.cube.position.set(-2,2,0)
            this.plane = new THREE.Mesh(new THREE.PlaneGeometry(5,32,32),new THREE.MeshLambertMaterial({color:"#333"}))
            this.plane.position.z = -0.5
            this.cube.castShadow = true;//开启阴影
            this.plane.receiveShadow = true;

            // new THREE.TextureLoader().load()
            this.sphere = new THREE.Mesh(new THREE.SphereGeometry(0.5), new THREE.MeshBasicMaterial({
                map: texture
            }))
            // this.sphere.position.set(0,0,0)
            // scale(-1, 1, 1)
            

            scene.add(this.cube)
            scene.add(this.plane)
            scene.add(this.sphere)
            container.appendChild(renderer.domElement)

        },
        addLight(){
            // 平行光
            const directionalLight = new THREE.PointLight( 0xffffff, 10 ,100);
            // const light = new THREE.HemisphereLight( 0xffffbb, 0x080820, 1 );
            directionalLight.position.set(-10,15,15)
            directionalLight.castShadow = true; // 设置平行光投射投影
            // directionalLight.target.position.set(-4, 0, -4)
            scene.add( directionalLight );

            const cameraHelper = new THREE.CameraHelper(directionalLight.shadow.camera)
            scene.add(cameraHelper)
        },
        render(){
            // requestAnimationFrame(this.render)
            // this.sphere.rotation.x += 0.005;
            // this.sphere.rotation.y += 0.005;

            renderer.render(scene,camera)
        },
    },
}
</script>

<style scoped>
.title{
    font-size: 24px;
    color: #ccc;
}
.container{
    width: 800px;
    height: 800px;
}
</style>