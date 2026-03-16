<template>
  <span class="badge" :class="`badge--${variant}`">
    <slot />
  </span>
</template>

<script setup lang="ts">
export type BadgeVariant = 'default' | 'outline'

withDefaults(defineProps<{
  variant?: BadgeVariant
}>(), {
  variant: 'default',
})
</script>

<style scoped>
.badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  border-radius: var(--radius-pill);
  font-size: 12px;
  font-weight: 500;
  font-family: var(--font-primary);
  line-height: 1;
  white-space: nowrap;
}

.badge--default {
  background: var(--accent);
  color: var(--accent-foreground);
}

.badge--outline {
  background: transparent;
  color: var(--foreground);
  border: 1px solid var(--border);
}
</style>
