<script setup lang="ts">
import { onMounted } from "vue"

defineProps({
  min: { type: [String, Number], default: 0 },
  max: { type: [String, Number], default: 100 },
  step: { type: [String, Number], default: 1 },
  modelValue: { type: [String, Number], default: 0 },
  label: { type: String, default: "" },
  subLabel: { type: String, default: "" },
})
defineEmits(["update:modelValue"])

const map = (
  current: number,
  inMin: number,
  inMax: number,
  outMin: number,
  outMax: number
): number => {
  return ((current - inMin) * (outMax - outMin)) / (inMax - inMin) + outMin
}

onMounted(() => {})
</script>
<template>
  <div class="py-2">
    <label for="large-range" class="flex justify-between mb-2 text-lg text-green-500">
      <span>{{ label }}</span>
      <span>{{ modelValue }}{{ subLabel }} </span>
    </label>
    <input
      id="large-range"
      type="range"
      :min="min"
      :max="max"
      :step="step"
      :value="modelValue"
      @input="$emit('update:modelValue', ($event.target as HTMLInputElement).value)"
      :style="{ 'background-size': `${map(+modelValue, +min, +max, 1, 100)}% 100%` }"
      class="w-full bg-gradient-to-r from-green-300 to-green-500 bg-gray-300 h-5 rounded-full appearance-none cursor-pointer bg-no-repeat"
    />
  </div>
</template>
<style lang="scss" scoped>
input[type="range"] {
  border-radius: 100px;

  &::-webkit-slider-thumb {
    -webkit-appearance: none;
    @apply w-10 h-10 bg-green-500;
    border-radius: 50%;
    cursor: ew-resize;
    box-shadow: 0 0 2px 0 #555;
    transition: background 0.3s ease-in-out;
  }
}
</style>
