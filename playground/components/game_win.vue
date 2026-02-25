<template>
  <v-dialog :model-value="modelValue" width="400" @update:model-value="val => emit('update:modelValue', val)">
    <v-card>
      <v-card-title class="text-h5">
        You Win!
      </v-card-title>
      <v-text-field
          v-model="username"
          label="Enter your name"
          width="200"
          class="ml-20"
        />
      <v-card-actions>
        <v-spacer />
        <v-btn color="green" variant="outlined" @click="submitRecord">
          Record
        </v-btn>
        <v-btn color="red" variant="flat" @click="handleRestart">
          Restart
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
<script setup lang="ts">
const props = defineProps<{
  modelValue: boolean,
  revealedCount: number,
  time: number
}>()

const username = ref("")
const emit = defineEmits(["update:modelValue", "restart", "record"])

function handleRestart() {
  emit('restart')
  emit('update:modelValue', false)
}
function submitRecord() {
  if (!username.value.trim()) return
  emit("record", username.value)
  username.value = ""
}
</script>

