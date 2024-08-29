<script setup lang="ts">
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import * as dat from 'dat.gui';

const experience = ref<HTMLCanvasElement | null>(null)

let renderer: any;
//let orbit;

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
scene.add(camera);

const axesHelper = new THREE.AxesHelper(5);
scene.add(axesHelper);

const gridHelper = new THREE.GridHelper(30, 20)
scene.add(gridHelper)

//camera.position.x = 1;
//camera.position.y = 1;
//camera.position.z = 5;
camera.position.set(-5, 10, 24);

// Figure Plain (floor)
const planeGeometry = new THREE.PlaneGeometry(30, 30);
const planeMaterial = new THREE.MeshStandardMaterial({
    color: 0xBBBBBB,
    side: THREE.DoubleSide
});
const plane = new THREE.Mesh(planeGeometry, planeMaterial);
plane.rotation.x = -0.5 * Math.PI
scene.add(plane)
plane.receiveShadow= true;

// Figure Box
const boxGeometry = new THREE.BoxGeometry();
const boxMaterial = new THREE.MeshBasicMaterial({color: 0x0FF00});
const box = new THREE.Mesh(boxGeometry, boxMaterial);
scene.add(box);

// Figure Sphere
const sphereGeometry = new THREE.SphereGeometry(3, 20, 20);
// MeshBasicMaterial / MeshStandardMaterial / MeshLambertMaterial
// const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0x000FFF, wireframe: false });
const sphereMaterial = new THREE.MeshStandardMaterial({ color: 0x000FFF, wireframe: false });
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
sphere.position.set(0, 5, 0)
scene.add(sphere);
sphere.castShadow = true;

// Light
const ambientLight = new THREE.AmbientLight(0x333333);
scene.add(ambientLight);
const directionalLight = new THREE.DirectionalLight(0xFFFFFF, 0.8);
scene.add(directionalLight);
directionalLight.position.set(-30, 50, 0)
directionalLight.castShadow = true;
directionalLight.shadow.camera.top = 10

const dLightHelper = new THREE.DirectionalLightHelper(directionalLight, 5);
scene.add(dLightHelper);
const dLightShadowHelper = new THREE.CameraHelper(directionalLight.shadow.camera);
scene.add(dLightShadowHelper);

// GUI
const gui = new dat.GUI();

const folder = gui.addFolder('Sphere');
const sphereOptions = {
    color: '#000FFF',
    wireframe: false ,
    speed: 0.01,
    segments: 10,
}
folder.addColor(sphereOptions, 'color').name('Color').onChange(function(e){
    sphere.material.color.set(e)
});
folder.add(sphereOptions, 'wireframe').name('Frame').onChange(function(e){
    sphere.material.wireframe = e
});
folder.add(sphereOptions, 'segments', 4, 50).name('Resolution').onChange(function(e){
    sphere.geometry.dispose()
    sphere.geometry = new THREE.SphereGeometry(3, e, e/2)
    //radius, widthSegments, heightSegments, phiStart, phiLength, thetaStart, thetaLength
});
folder.add(sphereOptions, 'speed', 0, 0.1).name('Animation speed')
folder.open();

// Animation
let step = 0;
function animate() {
    //box.rotation.x += 0.005;
    box.rotation.y += 0.01;
    //box.rotation.z += 0.005;

    step += sphereOptions.speed;
    sphere.position.y = 7 * Math.abs(Math.sin(step))

    renderer.render(scene, camera)
}


onMounted(() => {
    renderer = new THREE.WebGLRenderer({
        canvas: experience.value as unknown as HTMLCanvasElement,
        antialias: true,
    });
    renderer.shadowMap.enabled = true;
    renderer.setSize(window.innerWidth, window.innerHeight)
    renderer.setAnimationLoop(animate)

    const orbit = new OrbitControls(camera, renderer.domElement)
    orbit.update();
})

</script>

<template>
    <canvas ref="experience" />
</template>
