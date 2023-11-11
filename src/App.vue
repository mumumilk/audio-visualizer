<template>
  <div class="w-full min-h-screen bg-black grid place-items-center text-white">
    <button @click="toggle" class="z-50 absolute">
      {{ playing ? "pause" : "play" }}
    </button>
    <Bar :data-array="dataArray" />
  </div>
  <audio ref="audio" hidden src=""></audio>
</template>

<script setup lang="ts">
  import { ref, onMounted } from "vue";
  import Bar from "./components/Bar.vue";
  import Plane from "./components/Plane.vue";

  const SONGS_PATH = "/songs/";

  const audio = ref<HTMLAudioElement>();
  const playing = ref(false);

  const FFT_SIZE = 2048;
  const bufferLength = FFT_SIZE / 2;
  const dataArray = ref<Uint8Array>(new Uint8Array(bufferLength));

  let audioContext: AudioContext;

  function play() {
    audio.value?.play();
    playing.value = true;

    audioContext = new AudioContext();
    const analyser = audioContext.createAnalyser();
    analyser.fftSize = FFT_SIZE;

    const source = audioContext.createMediaElementSource(audio.value!);
    source.connect(analyser);
    analyser.connect(audioContext.destination);

    function draw() {
      requestAnimationFrame(draw);

      analyser.getByteFrequencyData(dataArray.value);
    }

    draw();
  }

  function pause() {
    audio.value?.pause();
    playing.value = false;
  }

  function toggle() {
    playing.value ? pause() : play();
  }
</script>

<style scoped>
  canvas {
    width: 100%;
    height: 100%;
  }
</style>
