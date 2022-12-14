<template>
    <div >
        <!-- <div ref="container" style="width:400px;height:400px;margin:0 auto;"></div> -->
        <div ref="heatmap" style="width:400px;height:400px;"></div>
    </div>
</template>

<script>
import * as THREE from "three";
import h337 from 'heatmap.js'
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

let renderer, scene, camera;
let heatmapInstance;
let texture;
let points;
let mesh;

const TemperatureColorStops = {
    1.0: "#f00",
    0.9: "#e2fa00",
    0.6: "#33f900",
    0.3: "#0349df",
    0.0: "#0f00ff",
};

export default {
    name: "twoDHeatmap",
    mounted() {
        this.init();
        this.animate();
    },
    methods: {
        init() {
            this.initRender();
            this.initCameraAndScene();
            this.initLight();
            this.addPluginHeatmap();
        },

        animate(){
            requestAnimationFrame(this.animate)
            renderer.render(scene, camera)
        },
        initRender() {
            // const container = this.$refs.container;
            const heatmap = this.$refs.heatmap;
            renderer = new THREE.WebGLRenderer({
                antialias: true,
            });
            renderer.setPixelRatio(window.devicePixelRatio);
            // console.log(container.clientWidth);
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setClearColor("#ccc");
            heatmap.appendChild(renderer.domElement);
        },

        initCameraAndScene() {
            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 1, 1000);
            camera.position.set(0, 0, 50);
            scene.add(camera);
        },

        initLight() {
            // 半球光HemisphereLight( skyColor : Integer, groundColor : Integer, intensity : Float )
            const hemiLight = new THREE.HemisphereLight("#fff", "#444", 0.5);
            hemiLight.position.set(0, 400, 0);
            scene.add(hemiLight);

            // 平行光DirectionalLight( color : Integer, intensity : Float )
            const dirLight = new THREE.DirectionalLight("#fff", 0.8);
            dirLight.position.set(0, 200, 100);
            // 设置平行光会产生动态阴影
            dirLight.castShadow = true;
            dirLight.shadow.camera.top = 180;
            dirLight.shadow.camera.bottom = -100;
            dirLight.shadow.camera.left = -120;
            dirLight.shadow.camera.right = 120;
            scene.add(dirLight);
        },

        addPluginHeatmap() {
            let that = this
            heatmapInstance = h337.create({
                container: that.$refs.heatmap,
                gradient: TemperatureColorStops,
                radius: 50,
                opacity: 0.5,
                blur: 0.8,
            })

            this.setHeatMapData();

            texture = new THREE.Texture(heatmapInstance._renderer.canvas);
            const material = new THREE.MeshLambertMaterial({
                map: texture,
                transparent: true,
                opacity: 1
            })

            mesh = new THREE.Mesh(new THREE.PlaneGeometry(10,10), material);
            // scene.add(mesh)

            if(texture){
                texture.needsUpdate = true
            }

        },

        // 设置热力图位置温度数据
        setHeatMapData(){
            points = [];
            let max = 0;
            const width = document.body.clientWidth;
            const height = document.body.clientHeight;
            let len = 500;
            // 随机位置点设置温度值
            while ( len -- ) {
                var val = Math.floor( Math.random() * 25 + 10 );
                max = Math.max( max, val );
                var point = {
                    x: Math.floor( Math.random() * width ),
                    y: Math.floor( Math.random() * height ),
                    value: val
                };
                points.push( point );
            }
            // 准备 heatmap 的数据
            const data = {
                max: max,
                data: points
            }

            heatmapInstance.setData( data );
        }
    },
};
</script>

<style>
</style>