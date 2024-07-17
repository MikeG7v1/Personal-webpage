<template>
  <div>
    <div class="canvas">
      <div ref="container"></div>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { useWindowSize } from "@vueuse/core";
import { computed } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { SpotLight } from "three";

var sphere = null;
var scene = null;
var camera = null;
var renderer = null;
var controls = null;

const { width, height } = useWindowSize();
const aspectRatio = computed(() => width.value / height.value);

export default {
  data() {
    return {};
  },
  mounted() {
    this.init();
    this.loop();
    window.addEventListener("resize", this.onWindowResize);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onWindowResize);
  },
  methods: {
    init() {
      // Define looks
      const geometry = new THREE.SphereGeometry(1, 32, 32);
      const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });

      // Object
      sphere = new THREE.Mesh(geometry, material);

      // Setting up the scene
      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(75, aspectRatio.value, 0.1, 100);
      camera.position.set(0, 0, 4);
      renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(window.devicePixelRatio);
      // renderer.shadowMap.enabled = true;
      this.$refs.container.appendChild(renderer.domElement);

      // Add OrbitControls
      controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true; // Enable inertia
      controls.dampingFactor = 0.25;
      controls.screenSpacePanning = false;
      controls.maxPolarAngle = Math.PI / 2;

      // Add objects and lights
      // scene.add(sphere);
      this.addScenery();
      this.addLight();
      this.setRgbeLoader();
    },
    onWindowResize() {
      this.updateCamera();
      this.updateRenderer();
    },
    updateCamera() {
      camera.aspect = aspectRatio.value;
      camera.updateProjectionMatrix();
    },
    updateRenderer() {
      renderer.setSize(width.value, height.value);
      renderer.render(scene, camera);
    },
    animate() {
      requestAnimationFrame(this.animate);
      renderer.render(scene, camera);
    },
    loop() {
      controls.update(); // Update controls on each frame
      this.updateRenderer();
      requestAnimationFrame(this.loop);
    },
    addScenery() {
      // const gltfLoader = new GLTFLoader();
      // gltfLoader.load(
      //   "scene3.glb",
      //   (gltf) => {
      //     const root = gltf.scene;
      //     scene.add(root);
      //     console.log("scene added", root);
      //   },
      //   (xhr) => {
      //     console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
      //   },
      //   (error) => {
      //     console.error(error);
      //   }
      // );
      // add arrows pointing at the scenery so I can find it easily
      // const arrowHelper = new THREE.ArrowHelper(
      //   new THREE.Vector3(0, 1, 0),
      //   new THREE.Vector3(0, 0, 0),
      //   2,
      //   0x00ff00
      // );
      // scene.add(arrowHelper);
    },

    addLight() {
      const light1 = new THREE.SpotLight(0xffffff);
      light1.position.set(100, 1000, 100);
      // light1.map = new THREE.TextureLoader().load("light.png");
      scene.add(light1);

      // const light2 = new THREE.Light(0xffffff, 1);
      // light2.position.set(-1, -1, -1);
      // scene.add(light2);

      // const light3 = new THREE.Light(0xffffff, 0.5);
      // scene.add(light3);
      // renderer.shadowMap.enabled = true;
      // renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      // light1.castShadow = true;
      // light2.castShadow = true;
      // const ambient = new THREE.AmbientLight(0xffffff, 0.5);
      // light.position.set(0, 0, 2);
      //scene.add(ambient);
    },
    setRgbeLoader() {
      const rgbeLoader = new RGBELoader();
      // rgbeLoader.setDataType(THREE.UnsignedByteType);
      const gltfLoader = new GLTFLoader();

      rgbeLoader.load("MR_INT-005_WhiteNeons_NAD.hdr", function (texture) {
        // const envMap = pmremGenerator.fromEquirectangular(texture).texture;
        // scene.background = envMap;
        // scene.environment = envMap;
        // texture.dispose();
        // pmremGenerator.dispose();
        texture.mapping = THREE.EquirectangularReflectionMapping;
        scene.environment = texture;
        gltfLoader.load(
          "scene3.glb",
          (gltf) => {
            const root = gltf.scene;
            scene.add(root);
            console.log("scene added", root);
          },
          (xhr) => {
            console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
          },
          (error) => {
            console.error(error);
          }
        );
      });
      renderer.outputEncoding = THREE.sRGBEncoding;
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      renderer.toneMappingExposure = 4;
    },
  },

  watch: {
    aspectRatio() {
      this.updateCamera();
      this.updateRenderer();
    },
  },
};
</script>

<style>
.canvas {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>
