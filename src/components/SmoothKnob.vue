<script setup>
import { computed, nextTick, onMounted, reactive } from "vue";

const props = defineProps({
  outerWidth: {
    type: String,
    required: false,
    default: "260px",
  },
  innerWidth: {
    type: String,
    required: false,
    default: "200px",
  },
  rate: {
    type: Number,
    required: true,
  },
  progressWidth: {
    type: Number,
    required: false,
    default: 5,
  },
  step: {
    type: Number,
    required: false,
    default: 5,
  },
});

const emits = defineEmits(["update:rate"]);

const outerWidthNum = computed(() => {
  return parseInt(props.outerWidth);
});

const currentRate = computed({
  get() {
    return props.rate;
  },
  set(val) {
    emits("update:rate", val);
  },
});

const rateRotateDeg = computed(() => {
  return `rotate(${360 * (currentRate.value / 100)}deg)`;
});

const radius = computed(() => {
  return outerWidthNum.value / 2 - props.progressWidth;
});

const circlePerimeter = computed(() => {
  return radius.value * 2 * Math.PI;
});

function plus(num) {
  const newVal = currentRate.value + (num || props.step);
  if (newVal >= 100) {
    currentRate.value = 100;
  } else {
    currentRate.value = newVal;
  }
}

function subtract(num) {
  const newVal = currentRate.value - (num || props.step);
  if (newVal <= 0) {
    currentRate.value = 0;
  } else {
    currentRate.value = newVal;
  }
}

defineExpose({ plus, subtract });

// move actions
const state = reactive({
  active: false,
  initialRatio: 0,
  initialX: 0,
  initialY: 0,
  centerX: 0,
  centerY: 0,
});

onMounted(() => {
  const oKnob = document.querySelector(".smooth-knob");

  const rect = oKnob.getBoundingClientRect();

  state.centerX = (rect.left + rect.right) / 2;
  state.centerY = (rect.top + rect.bottom) / 2;
});

function handleMouseDown(e) {
  state.active = true;
  state.initialX = e.x;
  state.initialY = e.y;
  state.initialRatio = currentRate.value;
}

function handleMouseUp() {
  state.active = false;
}

function handleMouseMove(e) {
  if (!state.active) return;

  const deltaX = e.x - state.centerX;
  const deltaY = e.y - state.centerY;
  const angle = Math.atan2(deltaY, deltaX);
  let degree = angle * (180 / Math.PI) + 90;

  if (degree <= -90) {
    degree += 360;
  }

  let newRate = Math.round((degree / 360) * 100);

  if (newRate < 0) {
    newRate = 100 + newRate;
  }

  currentRate.value = newRate;
}

function handleMouseLeave() {
  state.active = false;
}
</script>

<template>
  <div class="smooth-knob container">
    <div class="circle">
      <svg class="circle-svg">
        <circle
          class="circle-content"
          :cx="outerWidthNum / 2"
          :cy="outerWidthNum / 2"
          :r="radius"
          fill="none"
          stroke="#aaaaaa80"
          stroke-linecap="round"
          :stroke-width="props.progressWidth"
          :stroke-dasharray="`${
            (circlePerimeter * props.rate) / 100
          } ${circlePerimeter}`"
        ></circle>
      </svg>
    </div>

    <div
      class="knob"
      :class="{ active: state.active }"
      @mousedown="handleMouseDown"
      @mouseup="handleMouseUp"
      @mousemove="handleMouseMove"
      @mouseleave="handleMouseLeave"
    >
      <div class="tag"></div>
    </div>
  </div>
</template>

<style scoped lang="less">
.container {
  width: v-bind("props.outerWidth");
  height: v-bind("props.outerWidth");
  border-radius: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  // background: #000;
  position: relative;

  .circle {
    position: absolute;
    width: 100%;
    height: 100%;

    .circle-svg {
      width: 100%;
      height: 100%;
      transform: rotate(-90deg);

      circle {
        transition: all 0.1s;
      }
    }
  }

  .knob {
    width: v-bind("props.innerWidth");
    height: v-bind("props.innerWidth");
    border-radius: 100%;
    overflow: hidden;
    box-shadow: 5px 5px 10px #aaaaaa80, -5px -5px 10px #ffffff80;
    position: relative;
    transform: v-bind(rateRotateDeg);
    transition: 0.1s;

    &.active {
      box-shadow: 5px 5px 10px #aaaaaa30, -5px -5px 10px #ffffff30;
    }

    .tag {
      // background: #000;
      box-shadow: 5px 5px 10px #aaaaaa80 inset, -5px -5px 10px #ffffff80 inset;
      border-radius: 100%;
      width: 12%;
      height: 12%;
      z-index: 99;
      position: absolute;
      top: 5%;
      left: 50%;
      transform: translateX(-50%);
    }
  }
}
</style>
