<script setup lang="ts">
// TODO: scaleFactor를 조절할 수 있도록 하기.
// TODO: 각 요소의 스타일링을 제어할 수 있도록 만들기.
import { ref, computed, watch } from "vue";

const props = defineProps<{
  image: string;
  originImage: string;
  zoomLevel?: number;
}>();

const isActive = ref(false);
const squareElement = ref<HTMLDivElement | null>(null);
const imageElement = ref<HTMLDivElement | null>(null);
const originImageElement = ref<HTMLDivElement | null>(null);
const originImageWidth = ref(0);
const originImageHeight = ref(0);
const scaleFactor = ref(2);

const scaledOriginImageWidth = computed(
  () => originImageWidth.value * scaleFactor.value
);
const scaledOriginImageHeight = computed(
  () => originImageHeight.value * scaleFactor.value
);

const updateHandler = (e: MouseEvent) => {
  isActive.value = true;

  if (
    !squareElement.value ||
    !imageElement.value ||
    !originImageElement.value ||
    originImageWidth.value === 0 ||
    originImageHeight.value === 0
  ) {
    return;
  }

  const rect = imageElement.value.getBoundingClientRect();
  const squareWidth = squareElement.value.offsetWidth;
  const squareHeight = squareElement.value.offsetHeight;

  // 마우스 위치를 image 요소 내부로 제한
  const x = Math.min(
    Math.max(e.clientX - rect.left, squareWidth / 2),
    rect.width - squareWidth / 2
  );
  const y = Math.min(
    Math.max(e.clientY - rect.top, squareHeight / 2),
    rect.height - squareHeight / 2
  );

  // 네모의 위치 업데이트
  squareElement.value.style.left = `${x}px`;
  squareElement.value.style.top = `${y}px`;

  const ratioX = (x - squareWidth / 2) / (rect.width - squareWidth);
  const ratioY = (y - squareHeight / 2) / (rect.height - squareHeight);
  const bigImageX = ratioX * (scaledOriginImageWidth.value - rect.width);
  const bigImageY = ratioY * (scaledOriginImageHeight.value - rect.height);

  // 큰 이미지의 배경 위치 업데이트
  originImageElement.value.style.backgroundPosition = `-${bigImageX}px -${bigImageY}px`;
};

const leaveHandler = () => {
  isActive.value = false;
};

watch(
  () => props.originImage,
  (newOriginImage) => {
    if (!newOriginImage || newOriginImage === "") {
      return;
    }

    const image = new Image();
    image.onload = () => {
      originImageWidth.value = image.width;
      originImageHeight.value = image.height;
    };
    image.src = newOriginImage;
  },
  { immediate: true, deep: true }
);
</script>

<template>
  <div class="elevate-zoom-wrapper">
    <div
      class="current-image-container"
      ref="imageElement"
      @mousemove="updateHandler"
      @mouseleave="leaveHandler"
    >
      <img :src="image" />
      <div v-if="isActive" class="follower" ref="squareElement"></div>
    </div>
    <div
      v-if="isActive"
      class="origin-image-container"
      ref="originImageElement"
      :style="{
        background: `url(${originImage}) no-repeat 0 0`,
        backgroundSize: `${scaledOriginImageWidth}px ${scaledOriginImageHeight}px`,
      }"
    ></div>
  </div>
</template>

<style scoped>
.elevate-zoom-wrapper {
  position: relative;
  width: fit-content;
  height: 100%;
}
.current-image-container {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}
.current-image-container > img {
  width: 100%;
}
.follower {
  position: absolute;
  width: 10rem;
  height: 10rem;
  background-color: rgba(255, 255, 255, 0.5);
  pointer-events: none;
  cursor: none;
  transform: translate(-50%, -50%);
  border: 0.1rem solid #ccc;
}
.follower::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.origin-image-container {
  position: absolute;
  right: -100%;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 100;
  background-size: cover;
}
</style>
