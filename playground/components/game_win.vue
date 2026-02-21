<template>
  <v-dialog :model-value="modelValue" width="400" @update:model-value="emitClose">
    <v-card>
      <v-card-title class="text-h5">
        You Win!
      </v-card-title>
      <v-card-actions>
        <v-spacer />
        <v-btn color="red" variant="flat" @click="handleRestart">
          Restart
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
<script setup lang="ts">
const props = defineProps<{
  modelValue: boolean
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
  (e: 'restart'): void
}>()

function emitClose(value: boolean) {
  emit('update:modelValue', value)
}

function handleRestart() {
  emit('restart')
  emit('update:modelValue', false)
}
</script>

