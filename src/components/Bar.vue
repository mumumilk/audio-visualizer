<template>
  <div class="w-full h-screen grid place-items-center">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup lang="ts">
  import { onMounted, ref, watchEffect } from "vue";
  import { useResizeObserver } from "@vueuse/core";

  export interface BarProps {
    dataArray: Uint8Array;
  }

  const props = withDefaults(defineProps<BarProps>(), {
    dataArray: () => new Uint8Array(32),
  });

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

      ctx.fillStyle = "#020202";
      ctx.fillRect(0, 0, width.value, height.value);

      const slot = Math.abs(width.value / 32);
      const barWidth = slot * 0.94;

      for (let i = 0; i < 32; i++) {
        const barHeight = props.dataArray[i] + 5;
        const x = i * slot;
        const y = height.value - barHeight;
        ctx.beginPath();

        ctx.roundRect(x, y, barWidth, barHeight, [1, 1, 0, 0]);

        const gradient = ctx.createLinearGradient(
          i * barWidth,
          0,
          i * barWidth,
          height.value
        );
        gradient.addColorStop(0, "#fe8c00");
        gradient.addColorStop(1, "#f83600");

        ctx.fillStyle = gradient;
        ctx.fill();

        ctx.closePath();
      }
    }

    animate();
  });
</script>

<style scoped></style>
