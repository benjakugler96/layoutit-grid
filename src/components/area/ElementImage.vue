<template>
  <div
    :style="{
      ...style,
      position: 'relative',
    }"
  >
    <img
      width="500"
      height="500"
      :style="{ opacity: 0.5, width: '100%', height: '100%', 'object-fit': 'cover' }"
      :src="`https://picsum.photos/seed/${area.items ? area.type + '-' + item : area.name}/500?grayscale`"
    />
    <slot />
  </div>
</template>

<script setup>
import { defineProps } from 'vue'
defineProps({
  area: { type: Object, required: true },
  style: { type: Object, required: true },
})
</script>

<style scoped lang="scss"></style>
