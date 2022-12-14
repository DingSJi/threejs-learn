<template>
    <div>
        <div ref="container"></div>
        <div ref="info"></div>
    </div>
</template>

<script>
import * as THREE from "three";
import Stats from "three/addons/libs/stats.module.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { ImprovedNoise } from "three/addons/math/ImprovedNoise.js";

let container, stats;

let camera, controls, scene, renderer;

let mesh, texture;

const worldWidth = 256,
    worldDepth = 256,
    worldHalfWidth = worldWidth / 2,
    worldHalfDepth = worldDepth / 2;

let helper;
let raycaster = null,
    pointer = null;
export default {
    name: "TerrainRaycast",
    data() {
        return {
            data: "",
        };
    },
    mounted() {
        // 光线投射用于进行鼠标拾取（在三维空间中计算出鼠标移过了什么物体）
        raycaster = new THREE.Raycaster();
        // 表示2D vector（二维向量）的类
        pointer = new THREE.Vector2();

        this.init();
        this.animate();
    },
    methods: {
        init() {
            container = this.$refs.container;
            container.innerHTML = "";

            renderer = new THREE.WebGLRenderer({
                antialias: true, //是否执行抗锯齿。默认为false
            });
            // 设置设备像素比。通常用于避免HiDPI设备上绘图模糊
            renderer.setPixelRatio(window.devicePixelRatio);
            // 将输出canvas的大小调整为(width, height)并考虑设备像素比
            renderer.setSize(window.innerWidth, window.innerHeight);
            container.appendChild(renderer.domElement);

            scene = new THREE.Scene();
            scene.background = new THREE.Color(0xbfd1e5);

            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 10, 20000);

            // 轨道控制器 可以使得相机围绕目标进行轨道运动
            controls = new OrbitControls(camera, renderer.domElement);
            // 你能够将相机向内/外移动多少（仅适用于PerspectiveCamera），其默认值为0/Infinity
            controls.minDistance = 1000;
            controls.maxDistance = 10000;
            // 你能够垂直旋转的角度的上限，范围是0到Math.PI，其默认值为Math.PI
            controls.maxPolarAngle = Math.PI / 2;

            this.data = this.generateHeight(worldWidth, worldDepth);

            // 控制器的焦点，轨道围绕它运行
            controls.target.y = this.data[worldHalfWidth + worldHalfDepth * worldWidth] + 500;
            camera.position.y = controls.target.y + 2000;
            camera.position.x = 2000;
            // 更新控制器。必须在摄像机的变换发生任何手动改变后调用
            controls.update();

            // 平面缓冲几何体（PlaneGeometry）一个用于生成平面几何体的类width, height, widthSegments, heightSegments
            const geometry = new THREE.PlaneGeometry(7500, 7500, worldWidth - 1, worldDepth - 1);
            // 在 X 轴上旋转几何体
            geometry.rotateX(-Math.PI / 2);

            const vertices = geometry.attributes.position.array;

            for (let i = 0, j = 0, l = vertices.length; i < l; i++, j += 3) {
                vertices[j + 1] = this.data[i] * 10;
            }
            // console.log(vertices);

            // 从Canvas元素中创建纹理贴图
            texture = new THREE.CanvasTexture(this.generateTexture(this.data, worldWidth, worldDepth));
            // 定义了纹理贴图在水平/垂直方向上将如何包 裹默认值是THREE.ClampToEdgeWrapping
            texture.wrapS = THREE.ClampToEdgeWrapping;
            texture.wrapT = THREE.ClampToEdgeWrapping;

            // Mesh( geometry : BufferGeometry, material : Material )
            // MeshBasicMaterial 基础网格材质 一个以简单着色（平面或线框）方式来绘制几何体的材质。这种材质不受光照的影响。map : Texture颜色贴图
            mesh = new THREE.Mesh(
                geometry,
                new THREE.MeshBasicMaterial({
                    map: texture,
                })
            );
            scene.add(mesh);

            // 圆锥缓冲几何体（ConeGeometry）一个用于生成圆锥几何体的类。radius — 圆锥底部的半径，默认值为1。height — 圆锥的高度，默认值为1。radialSegments — 圆锥侧面周围的分段数，默认为8。
            const geometryHelper = new THREE.ConeGeometry(20, 100, 3);
            geometryHelper.translate(0, 50, 0);
            geometryHelper.rotateX(Math.PI / 2);
            // 法线网格材质(MeshNormalMaterial) 一种把法向量映射到RGB颜色的材质。
            helper = new THREE.Mesh(geometryHelper, new THREE.MeshNormalMaterial());
            scene.add(helper);

            container.addEventListener("pointermove", this.onPointerMove);

            // 显示帧数
            stats = new Stats();
            container.appendChild(stats.dom);

            window.addEventListener('resize',this.onWindowResize)
        },

        onWindowResize(){
            camera.aspect = window.innerWidth / window.innerHeight;
            // 手动更新相机的投影矩阵
            camera.updateProjectionMatrix();

            renderer.setSize(window.innerWidth, window.innerHeight);
        },

        animate(){
            requestAnimationFrame(this.animate);

            this.render();
            stats.update();
        },

        render(){
            renderer.render(scene, camera)
        },

        generateHeight(width, height) {
            const size = width * height,
                data = new Uint8Array(size),
                // Perlin函数，它常用在模拟自然物体的地方，比如地形，海水等
                // 要构造一个Perlin函数，你首先需要一个噪声函数和一个插值函数
                perlin = new ImprovedNoise(),
                z = Math.random() * 100;
            let quality = 1;
            for (let j = 0; j < 4; j++) {
                for (let i = 0; i < size; i++) {
                    const x = i % width,
                        y = ~~(i / width);
                    data[i] += Math.abs(perlin.noise(x / quality, y / quality, z) * quality * 1.75);
                }

                quality *= 5;
            }

            return data;
        },

        generateTexture(data, width, height) {
            // bake lighting into texture
            let context, image, imageData, shade;

            // 三维向量
            const vector3 = new THREE.Vector3(0, 0, 0);
            const sun = new THREE.Vector3(1, 1, 1);
            // 将该向量转换为单位向量（unit vector）， 也就是说，将该向量的方向设置为和原向量相同，但是其长度（length）为1。
            sun.normalize();

            const canvas = document.createElement("canvas");
            canvas.width = width;
            canvas.height = height;
            context = canvas.getContext("2d");
            context.fillStyle = "#000";
            context.fillRect(0, 0, width, height);
            // getImageData返回一个 ImageData 对象，用来描述 canvas 区域隐含的像素数据
            image = context.getImageData(0, 0, canvas.width, canvas.height);
            imageData = image.data;
            for (let i = 0, j = 0, l = imageData.length; i < l; i += 4, j++) {
                vector3.x = data[j - 2] - data[j + 2];
                vector3.y = 2;
                vector3.z = data[j - width * 2] - data[j + width * 2];
                vector3.normalize();

                shade = vector3.dot(sun);

                imageData[i] = (96 + shade * 128) * (0.5 + data[j] * 0.007);
                imageData[i + 1] = (32 + shade * 96) * (0.5 + data[j] * 0.007);
                imageData[i + 2] = shade * 96 * (0.5 + data[j] * 0.007);
            }
            context.putImageData(image, 0, 0);

            // Scaled 4x
            const canvasScaled = document.createElement("canvas");
            canvasScaled.width = width * 4;
            canvasScaled.height = height * 4;

            context = canvasScaled.getContext("2d");
            context.scale(4, 4);
            context.drawImage(canvas, 0, 0);

            image = context.getImageData(0, 0, canvasScaled.width, canvasScaled.height);
            imageData = image.data;

            for (let i = 0, l = imageData.length; i < l; i += 4) {
                const v = ~~(Math.random() * 5);

                imageData[i] += v;
                imageData[i + 1] += v;
                imageData[i + 2] += v;
            }

            context.putImageData(image, 0, 0);

            return canvasScaled;
        },

        onPointerMove(event) {
            pointer.x = (event.clientX / renderer.domElement.clientWidth) * 2 - 1;
            pointer.y = -(event.clientY / renderer.domElement.clientHeight) * 2 + 1;
            // setFromCamera ( coords : Vector2, camera : Camera )
            raycaster.setFromCamera(pointer, camera);

            // 看看照相机的射线会不会射到我们的网格上
            const intersects = raycaster.intersectObject(mesh);

            // 为我们点击的网格切换旋转图标
            if (intersects.length > 0) {
                helper.position.set(0, 0, 0);
                helper.lookAt(intersects[0].face.normal);

                helper.position.copy(intersects[0].point);
            }
        },

        
    },
};
</script>

<style scoped>
body {
    background-color: #bfd1e5;
    color: #61443e;
}
</style>