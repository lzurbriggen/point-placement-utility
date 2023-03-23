<template>
  <div :class="$style.view">
    <input type="file" @change="imageChanged" />
    <div>
      {{ percentagePos }}
    </div>
    <div v-if="image" :class="$style.imgContainer">
      <img
        ref="imgRef"
        :class="[$style.img, { [$style.pressing]: pressing }]"
        draggable="false"
        :src="image"
        @mousedown="
          pressing = true;
          setTargetPos();
        "
        @mousemove="setTargetPos"
      />
      <div :class="$style.target" :style="targetStyle"></div>
    </div>

    <div>
      {{ targetPos }}
    </div>
  </div>
</template>

<script lang="ts" setup>
import { useMouseInElement } from "@vueuse/core";
import type { CSSProperties } from "vue";
import { computed, ref, watchEffect } from "vue";

export type Vec = [number, number];

const imgRef = ref<HTMLImageElement>();
const image = ref("");
const pressing = ref(false);
const targetPos = ref<Vec>([0, 0]);

const targetStyle = computed<CSSProperties>(() => {
  return {
    left: `${targetPos.value[0]}px`,
    top: `${targetPos.value[1]}px`,
  };
});

const { elementX, elementY, elementWidth, elementHeight } = useMouseInElement(imgRef);

const clamp = (val: number, min: number, max: number) => {
  return Math.min(Math.max(val, min), max);
};

const setTargetPos = () => {
  if (!pressing.value) {
    return;
  }
  targetPos.value = [clamp(elementX.value, 0, elementWidth.value), clamp(elementY.value, 0, elementHeight.value)];
};

document.addEventListener("mousemove", setTargetPos);

document.addEventListener("mouseup", () => {
  pressing.value = false;
  setTargetPos();
});

const percentagePos = computed<Vec>(() => [
  targetPos.value[0] / elementWidth.value,
  targetPos.value[1] / elementHeight.value,
]);

watchEffect(() => {
  navigator.clipboard.writeText(JSON.stringify(percentagePos.value));
});

const imageChanged = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const files = target.files;

  if (files?.[0]) {
    const reader = new FileReader();
    reader.onload = () => {
      image.value = reader.result as string;
    };
    reader.readAsDataURL(files[0]);
  }
};
</script>

<style module lang="scss">
.view {
  padding: 1rem;
  width: 100vw;
  min-height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.imgContainer {
  position: relative;
  max-width: 100%;

  .img {
    width: 100%;
    cursor: crosshair;

    &.pressing {
      cursor: none;
    }
  }

  .target {
    position: absolute;
    width: 0;
    height: 0;
    pointer-events: none;
    --target-size: 12px;

    &::after {
      display: block;
      box-sizing: border-box;
      content: "";
      position: relative;
      top: calc(var(--target-size) / 2 * -1);
      left: calc(var(--target-size) / 2 * -1);
      width: var(--target-size);
      height: var(--target-size);
      border: 2px solid red;
      border-radius: 100%;
    }
  }
}
</style>
