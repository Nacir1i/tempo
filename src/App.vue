<script setup lang="ts">
import {
  rand,
  useTransition,
  TransitionPresets,
  useElementBounding,
} from "@vueuse/core";
import { computed, ref, shallowRef, useTemplateRef } from "vue";

type Step = "FIRST" | "SECOND";

const STEP_MAP: Record<number, Step> = {
  1: "FIRST",
  2: "SECOND",
};

const duration = 300;

const targetOffset = shallowRef([0, 0]);
const offset = useTransition(targetOffset, {
  duration,
  transition: TransitionPresets.easeOutExpo,
});

const currentStepIndex = ref(1);
const currentStep = computed(() => {
  return STEP_MAP[currentStepIndex.value];
});

const page = useTemplateRef("page");
const noButton = useTemplateRef("noButton");
const pageBounds = useElementBounding(page);
const noButtonBounds = useElementBounding(noButton);

function nextStep() {
  const keys = Object.keys(STEP_MAP).map((stringNumber) =>
    parseInt(stringNumber),
  );

  const sortedKeys = keys.sort((a, b) => b - a);
  const biggestIndex = sortedKeys.at(0);

  if (biggestIndex && currentStepIndex.value === biggestIndex) return;

  currentStepIndex.value += 1;
}

function clamp(value: number, min: number, max: number) {
  return Math.min(Math.max(value, min), max);
}

function rejectHover() {
  const movement = 100;

  const nextX = targetOffset.value[0] + rand(-movement, movement);
  const nextY = targetOffset.value[1] + rand(-movement, movement);

  const currentX = targetOffset.value[0];
  const currentY = targetOffset.value[1];

  const minX = pageBounds.left.value - noButtonBounds.left.value + currentX;
  const maxX = pageBounds.right.value - noButtonBounds.right.value + currentX;
  const minY = pageBounds.top.value - noButtonBounds.top.value + currentY;
  const maxY = pageBounds.bottom.value - noButtonBounds.bottom.value + currentY;

  targetOffset.value = [clamp(nextX, minX, maxX), clamp(nextY, minY, maxY)];
}
</script>

<template>
  <div
    ref="page"
    class="w-screen h-screen flex justify-center items-center bg-pink-200"
  >
    <div
      v-if="currentStep === 'FIRST'"
      class="flex gap-4 bg-pink-300 rounded-lg border border-pink-600 p-3 flex-col"
    >
      <p>Hello meryem ❤️</p>

      <p>Do you like me ?</p>

      <div class="flex gap-4 flex-1">
        <button
          class="p-1 bg-green-300 rounded-lg border border-green-700"
          @click="nextStep"
        >
          Yes, indeed
        </button>
        <button
          ref="noButton"
          class="p-1 bg-red-300 rounded-lg border border-red-700"
          @mouseenter="rejectHover"
          @click="rejectHover"
          :style="{ transform: `translate(${offset[0]}px, ${offset[1]}px)` }"
        >
          No
        </button>
      </div>
    </div>

    <img
      v-if="currentStep === 'SECOND'"
      src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExNzJlbHEyNmJnc25mNDg1aGFwaHFqdGl2YWFiZGEzc25rd3I1Z241NiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/zNbiX43QsqUAU/giphy.gif"
      alt="damnright"
    />
  </div>
</template>
