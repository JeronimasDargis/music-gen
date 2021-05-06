<template>
  <div>
    <Input @audio-upload="initWave()" />
  </div>
</template>

<script>
import * as THREE from "three";
import * as TWEEN from "@tweenjs/tween.js";
import Input from "./components/Input";
import { TrackballControls } from "../static/TrackballControls.js";

export default {
  data: () => ({
    backgroundColor: "#a9a9a9",
    ambientLightColor: 0xffffff,
    spotLightColor: 0xffffff,
    boxColor: 0x1a63ed,
    angle: 0,
    waveCol: 0,
    waveRow: 0,
    gridSize: 50,
    mesh: null,
    boxes: [],
    wave: {
      velocity: 0.05,
      amplitude: -20,
      waveLength: 242,
    },
    frequency: 0,
    scene: null,
    camera: null,
    renderer: null,
    spotLight: null,
    controls: null,
  }),
  components: {
    Input,
  },
  name: "App",

  mounted() {
    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color(this.backgroundColor);
    this.camera = new THREE.PerspectiveCamera(
      20,
      window.innerWidth / window.innerHeight,
      1,
      1000
    );
    this.camera.position.set(-100, 100, -100);

    this.addRenderer();

    document.body.appendChild(this.renderer.domElement);

    this.controls = new TrackballControls(
      this.camera,
      this.renderer.domElement
    );

    // this.controls.enableDamping = true;
    // this.controls.dampingFactor = 0.04;

    this.controls.addEventListener("start", () => {
      requestAnimationFrame(() => {
        document.body.style.cursor = "-moz-grabbing";
        document.body.style.cursor = "-webkit-grabbing";
      });
    });

    this.controls.addEventListener("end", () => {
      requestAnimationFrame(() => {
        document.body.style.cursor = "-moz-grab";
        document.body.style.cursor = "-webkit-grab";
      });
    });

    this.addAmbientLight();

    this.addDirectionalLight();

    this.addFloor();

    this.addBoxes();

    // this.addGUIControls();

    this.animate();

    window.addEventListener("resize", this.onWindowResize, false);
  },

  methods: {
    addDirectionalLight() {
      this.directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      this.directionalLight.castShadow = true;
      this.directionalLight.position.set(0, 1, 0);

      this.directionalLight.shadow.camera.far = 1000;
      this.directionalLight.shadow.camera.near = -100;

      this.directionalLight.shadow.camera.left = -40;
      this.directionalLight.shadow.camera.right = 40;
      this.directionalLight.shadow.camera.top = 20;
      this.directionalLight.shadow.camera.bottom = -20;
      this.directionalLight.shadow.camera.zoom = 1;
      this.directionalLight.shadow.camera.needsUpdate = true;

      const targetObject = new THREE.Object3D();
      targetObject.position.set(-50, -82, 40);
      this.directionalLight.target = targetObject;

      this.scene.add(this.directionalLight);
      this.scene.add(this.directionalLight.target);
    },

    addRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    addAmbientLight() {
      const light = new THREE.AmbientLight(this.ambientLightColor, 0.5);
      this.scene.add(light);
    },
    addSpotLight() {
      this.spotLight = new THREE.SpotLight(this.spotLightColor);
      this.spotLight.position.set(100, 250, 150);
      this.spotLight.castShadow = true;
      this.scene.add(this.spotLight);
    },
    clearScene() {
      this.scene.remove(this.mesh);

      this.boxes = [];
    },
    addBoxes() {
      const col = this.gridSize;
      const row = this.gridSize;
      const size = 1;
      const height = 5;
      const material = new THREE.MeshLambertMaterial({
        color: this.boxColor,
      });

      const geometry = new THREE.BoxBufferGeometry(size, height, size);
      geometry.translate(0, 2.5, 0);
      this.mesh = this.getBox(geometry, material, row * col);
      this.scene.add(this.mesh);

      let ii = 0;

      for (let i = 0; i < col; i++) {
        this.boxes[i] = [];

        for (let j = 0; j < row; j++) {
          const pivot = new THREE.Object3D();
          this.boxes[i][j] = pivot;

          pivot.scale.set(1, 0.001, 1);
          pivot.position.set(
            i - this.gridSize * 0.5,
            height * 0.5,
            j - this.gridSize * 0.5
          );

          pivot.updateMatrix();
          this.mesh.setMatrixAt(ii++, pivot.matrix);
        }
      }

      this.mesh.instanceMatrix.needsUpdate = true;
    },
    drawWave() {
      const col = this.gridSize;
      const row = this.gridSize;
      let ii = 0;

      for (let i = 0; i < col; i++) {
        for (let j = 0; j < row; j++) {
          const distance = this.distance(j, i, row * 0.5, col * 0.5);

          const offset = this.map(distance, 0, this.wave.waveLength, -100, 100);
          const angle = this.angle + offset;
          this.boxes[i][j].scale.y = this.map(
            Math.sin(angle),
            -1,
            -this.wave.amplitude,
            0.001,
            1
          );

          this.boxes[i][j].updateMatrix();
          this.mesh.setMatrixAt(ii++, this.boxes[i][j].matrix);
        }
      }

      this.mesh.instanceMatrix.needsUpdate = true;

      this.angle -= this.wave.velocity;
    },
    initWave() {
      // this.gridSize = 100;

      new TWEEN.Tween(this.wave)
        .to(
          {
            velocity: 0.1,
            amplitude: -1,
          },
          5000
        )
        .start();

      setTimeout(() => {
        new TWEEN.Tween(this.wave)
          .to(
            {
              waveLength: 100,
              amplitude: -5,
            },
            5000
          )
          .start();
      }, 5000);
      setTimeout(() => {
        new TWEEN.Tween(this.wave)
          .to(
            {
              waveLength: 244,
              velocity: 0.3,
              amplitude: -1,
            },
            5000
          )
          .start();
      }, 10000);
    },
    distance(x1, y1, x2, y2) {
      return Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
    },

    map(value, start1, stop1, start2, stop2) {
      return ((value - start1) / (stop1 - start1)) * (stop2 - start2) + start2;
    },

    addFloor() {
      const planeGeometry = new THREE.PlaneBufferGeometry(500, 500);
      const planeMaterial = new THREE.ShadowMaterial({ opacity: 0.35 });

      this.floor = new THREE.Mesh(planeGeometry, planeMaterial);

      planeGeometry.rotateX(-Math.PI / 2);

      this.floor.position.y = 2;
      this.floor.receiveShadow = true;

      this.scene.add(this.floor);
    },
    getBox(geometry, material, count) {
      const mesh = new THREE.InstancedMesh(geometry, material, count);
      mesh.instanceMatrix.setUsage(THREE.DynamicDrawUsage);
      mesh.castShadow = true;
      mesh.receiveShadow = true;

      return mesh;
    },

    addGrid() {
      const size = this.col;
      const divisions = size;
      const gridHelper = new THREE.GridHelper(size, divisions);

      gridHelper.position.set(0, 0, 0);
      gridHelper.material.opacity = 0;
      gridHelper.material.transparent = true;

      this.scene.add(gridHelper);
    },
    animate() {
      requestAnimationFrame(this.animate);
      TWEEN.update();
      this.drawWave();
      this.controls.update();
      this.renderer.render(this.scene, this.camera);
    },

    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.animate();
    },
  },
};
</script>

<style>
body {
  font-family: sans-serif;
  font-size: 11px;
  background-color: #000;
  margin: 0px;
}
canvas {
  display: block;
}
</style>
