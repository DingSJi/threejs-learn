<template>
    <div>
        <div ref="heatmap" style="width: 256px;height: 256px;display: none"></div>
        <div id="map" ref="map"></div>
    </div>
</template>

<script>
import * as THREE from "three";
import * as maptalks from "maptalks";
import h337 from 'heatmap.js'
// import { ThreeLayer } from 'maptalks.three/dist/maptalks.three.min.js';
import { TrackballControls } from "three/addons/controls/TrackballControls.js";
import "maptalks/dist/maptalks.css";

let camera, scene, renderer, container, controls;
export default {
    name: "threeDHeatmap",
    mounted() {
        this.init();
    },
    methods: {
        init() {
            let that = this
            var heatmapInstance = h337.create({
                // only container is required, the rest will be defaults
                container: that.$refs.heatmap,
            });

            // now generate some random data
            var points = [];
            var max = 0;
            var width = 256;
            var height = 256;
            var len = 20;

            while (len--) {
                var val = Math.floor(Math.random() * 100);

                max = Math.max(max, val);
                var point = {
                    x: Math.floor(Math.random() * width),
                    y: Math.floor(Math.random() * height),
                    value: val,
                };
                points.push(point);
            }

            // heatmap data format
            var data = {
                max: max,
                data: points,
            };

            // if you have a set of datapoints always use setData instead of addData
            // for data initialization
            heatmapInstance.setData(data);

            var map = new maptalks.Map("map", {
                center: [13.416935229170008, 52.529564137540376],
                zoom: 15,
                fog: false,
                maxPitch: 60,
                pitch: 60,
                centerCross: false,
                doubleClickZoom: false,
                attribution: null,
                baseLayer: new maptalks.TileLayer("tile", {
                    urlTemplate: "https://{s}.basemaps.cartocdn.com/dark_nolabels/{z}/{x}/{y}.png",
                    subdomains: ["a", "b", "c", "d"],
                }),
            });

            var threeLayer = new maptalks.ThreeLayer("t", {
                forceRenderOnMoving: true,
                forceRenderOnRotating: true,
            }).addTo(map);
            let planeLoaded = false;

            threeLayer.prepareToDraw = function (gl, scene, camera) {
                var me = this;
                var light = new THREE.DirectionalLight(0xffffff);
                light.position.set(0, 0, 1);
                scene.add(light);

                var geometry = new THREE.PlaneGeometry(0, 0, 255, 255);

                let geolen = geometry.vertices.length;

                var mdata;
                for (let i = 0; i < geolen; i++) {
                    geometry.vertices[i].z = heatmapInstance.getValueAt({ x: i % 256, y: Math.trunc(i / 256) }) / (256 * 2);
                }

                let texture = new THREE.CanvasTexture(heatmapInstance._renderer.canvas);

                var material = new THREE.MeshBasicMaterial({ map: texture, transparent: true, side: THREE.DoubleSide, depthTest: true });
                var plane = new THREE.Mesh(geometry, material);
                //plane.rotateX( - Math.PI / 2 );
                plane.scale.set(10, 10, 10);
                var v = threeLayer.coordinateToVector3(new maptalks.Coordinate(map.getCenter()));
                plane.position.x = v.x;
                plane.position.y = v.y;
                plane.position.z = v.z;
                scene.add(plane);
                planeLoaded = true;
            };
        },
    },
};
</script>

<style>
#map {
    width: 100%;
    height: 100%;
    background-color: #000;
}
</style>