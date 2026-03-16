<template>
  <component
    :is="as"
    class="chip"
    :class="{
      'chip--icon': icon,
      'chip--active': active
    }"
    @click="onClick"
  >
    <component
      v-if="icon"
      :is="icon"
      :size="14"
      class="chip-icon"
    />
    <span class="chip-label"><slot /></span>
  </component>
</template>

<script setup lang="ts">
import type { Component } from 'vue'

const props = withDefaults(defineProps<{
  as?: 'button' | 'span'
  icon?: Component
}>(), {
  as: 'span',
})

const active = defineModel<boolean>('active')

function onClick() {
  if (props.as !== 'button') return
  active.value = !active.value
}
</script>

<style scoped>
.chip {
  display: inline-flex;
  align-items: center;
  padding: 6px 14px;
  border-radius: var(--radius-pill);
  background-color: var(--secondary);
  color: var(--foreground);
  font-size: 13px;
  font-weight: 500;
  border: none;
  outline: none;
  cursor: default;
  font-family: inherit;
}

button.chip {
  cursor: pointer;
}

.chip--icon {
  padding: 8px 14px;
  gap: 6px;
}

.chip--active {
  background-color: var(--primary);
  color: var(--primary-foreground);
}
</style>
