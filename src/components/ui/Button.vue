<template>
  <button
    class="btn"
    :class="[`btn--${variant}`]"
  >
    <component
      v-if="icon"
      :is="icon"
      :size="16"
      class="btn-icon"
    />
    <span
      v-if="$slots.default"
      class="btn-label"
    >
      <slot />
    </span>
  </button>
</template>

<script setup lang="ts">
import type { Component } from 'vue'

export type ButtonVariant = 'primary' | 'secondary' | 'outline' | 'ghost'

withDefaults(defineProps<{
  variant?: ButtonVariant
  icon?: Component
}>(), {
  variant: 'primary',
})
</script>

<style scoped>
.btn {
  padding: 10px 20px;
  display: flex;
  align-items: center;
  gap: 8px;
  border-radius: var(--radius-m);
  border: none;
  outline: none;
  font-size: 14px;
}

.btn:hover {
  cursor: pointer;
}

.btn--primary {
  background-color: var(--primary);
  color: var(--primary-foreground);
}
</style>
