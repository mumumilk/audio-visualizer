<template>
  <div ref="container" class="w-full h-screen grid place-items-center"></div>
</template>

<script setup lang="ts">
  import { computed, onMounted, ref, watchEffect } from "vue";
  import { useResizeObserver } from "@vueuse/core";
  import {
    Scene,
    PerspectiveCamera,
    WebGLRenderer,
    PlaneGeometry,
    ShaderMaterial,
    Mesh,
  } from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import { vertexShader, fragmentShader } from "./shaders";

  export interface PlaneProps {
    dataArray: Uint8Array;
  }

  const props = withDefaults(defineProps<PlaneProps>(), {
    dataArray: () => new Uint8Array(32),
  });

  const size = computed(() => props.dataArray.length);

  const container = ref<HTMLDivElement | null>(null);

  onMounted(() => {
    if (!container.value) return;

    const scene = new Scene();
    const camera = new PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    const renderer = new WebGLRenderer();
    renderer.setSize(
      container.value!.clientWidth,
      container.value!.clientHeight
    );
    container.value.appendChild(renderer.domElement);

    let uniforms = {
      u_time: {
        type: "f",
        value: 1.0,
      },
      u_amplitude: {
        type: "f",
        value: 3.0,
      },
      u_data_arr: {
        type: "float[64]",
        value: props.dataArray,
      },
    };

    // Create the plain geometry
    const geometry = new PlaneGeometry(64, 64, 64, 64);

    // Create the material
    const material = new ShaderMaterial({
      // note: this is where the magic happens
      uniforms: uniforms,
      vertexShader: vertexShader(),
      fragmentShader: fragmentShader(),
      wireframe: true,
    });

    // Create the mesh and add it to the scene
    const mesh = new Mesh(geometry, material);
    mesh.rotation.x = -Math.PI / 2 + Math.PI / 4;
    mesh.scale.x = 2;
    mesh.scale.y = 2;
    mesh.scale.z = 2;
    mesh.position.y = 8;
    scene.add(mesh);

    // Add orbital controls
    const controls = new OrbitControls(camera, renderer.domElement);
    const speed = 0.8;
    controls.zoomSpeed = speed;
    controls.autoRotate = true;
    controls.autoRotateSpeed = 0.5;
    controls.panSpeed = speed;
    controls.rotateSpeed = speed;
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;

    camera.position.set(0, 0, 100);
    controls.update();

    // const heightRatio = 0.4;
    // const widthRatio = 0.8;

    // let width = ref(canvas.value.parentElement!.clientWidth * widthRatio);
    // let height = ref(canvas.value.parentElement!.clientHeight * heightRatio);

    // watchEffect(() => (canvas.value!.width = width.value));
    // watchEffect(() => (canvas.value!.height = height.value));

    // useResizeObserver(canvas.value.parentElement, (entries) => {
    //   const entry = entries[0];
    //   width.value = entry.contentRect.width * widthRatio;
    //   height.value = entry.contentRect.height * heightRatio;
    // });

    function animate(time: number) {
      requestAnimationFrame(animate);

      uniforms.u_time.value = time;
      uniforms.u_data_arr.value = props.dataArray;

      controls.update();
      renderer.render(scene, camera);
    }

    animate(0);
  });
</script>

<style scoped></style>
