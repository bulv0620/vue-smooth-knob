<script setup>
import { ref } from "vue";
import SmoothKnob from "./components/SmoothKnob.vue";

const rate = ref(10);

const knobRef = ref(null);

let timeoutTimer = null;
let intervalTimer = null;

function handlePlus() {
  knobRef.value.plus(1);

  timeoutTimer = setTimeout(() => {
    intervalTimer = setInterval(() => {
      knobRef.value.plus(1);
    }, 50);
  }, 500);
}

function handleSubtract() {
  knobRef.value.subtract(1);

  timeoutTimer = setTimeout(() => {
    intervalTimer = setInterval(() => {
      knobRef.value.subtract(1);
    }, 50);
  }, 500);
}

function handleStop() {
  if (timeoutTimer) {
    clearTimeout(timeoutTimer);
    timeoutTimer = null;
  }

  if (intervalTimer) {
    clearInterval(intervalTimer);
    intervalTimer = null;
  }
}
</script>

<template>
  <div class="main">
    <h1>{{ `${rate}%` }}</h1>
    <SmoothKnob ref="knobRef" v-model:rate="rate" />
    <div class="btns">
      <button class="btn" @mousedown="handlePlus" @mouseup="handleStop">
        +
      </button>
      <button class="btn" @mousedown="handleSubtract" @mouseup="handleStop">
        -
      </button>
    </div>
  </div>
</template>

<style scoped>
.main {
  height: 100vh;
  color: #606266;
  display: flex;
  gap: 18px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #e3e3e3;

  .btns {
    display: flex;
    gap: 18px;

    .btn {
      border: none;
      background: transparent;
      box-shadow: 5px 5px 10px #aaaaaa80, -5px -5px 10px #ffffff80;
      padding: 8px 16px;
      width: 60px;
      border-radius: 12px;
      color: #606266;
      font-size: 24px;

      &:active {
        box-shadow: 5px 5px 10px #aaaaaa80 inset, -5px -5px 10px #ffffff80 inset;
      }
    }
  }
}
</style>
