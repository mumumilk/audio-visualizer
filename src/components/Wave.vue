<template>
  <div class="w-full h-screen grid place-items-center">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup lang="ts">
  import { computed, onMounted, ref, watchEffect } from "vue";
  import { useResizeObserver } from "@vueuse/core";

  export interface WaveProps {
    dataArray: Uint8Array;
  }

  const props = withDefaults(defineProps<WaveProps>(), {
    dataArray: () => new Uint8Array(32),
  });

  const size = computed(() => props.dataArray.length);

  const canvas = ref<HTMLCanvasElement | null>(null);

  onMounted(() => {
    if (!canvas.value) return;

    const heightRatio = 0.4;
    const widthRatio = 0.8;

    let width = ref(canvas.value.parentElement!.clientWidth * widthRatio);
    let height = ref(canvas.value.parentElement!.clientHeight * heightRatio);

    watchEffect(() => (canvas.value!.width = width.value));
    watchEffect(() => (canvas.value!.height = height.value));

    useResizeObserver(canvas.value.parentElement, (entries) => {
      const entry = entries[0];
      width.value = entry.contentRect.width * widthRatio;
      height.value = entry.contentRect.height * heightRatio;
    });

    const ctx = canvas.value.getContext("2d");

    function animate(time?: number) {
      requestAnimationFrame(animate);

      if (!ctx) return;

      // reset canvas
      ctx.fillStyle = "black";
      ctx.fillRect(0, 0, width.value, height.value);

      const sliceWidth = width.value / size.value;

      ctx.lineWidth = 1;

      let x = 0;

      ctx.beginPath();
      ctx.moveTo(x, height.value);

      for (let i = 0; i < size.value; i++) {
        const v = props.dataArray[i] / 128.0;
        const y = v * (height.value / 2);

        if (i === 0) {
          ctx.lineTo(x, y);
        }

        ctx.lineTo(x, y);
        x += sliceWidth;

        if (i === size.value - 1) {
          ctx.lineTo(x, y);
          ctx.lineTo(x, height.value);
          ctx.lineTo(0, height.value);
        }

        const gradient = ctx.createLinearGradient(x, 0, x, height.value / 2);
        gradient.addColorStop(0, "#fe8c00");
        gradient.addColorStop(1, "#f83600");

        ctx.fillStyle = gradient;
        ctx.fill();
      }
    }

    animate();
  });
</script>

<style scoped></style>
