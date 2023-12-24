<template>
  <v-dialog>
    <template #activator="{ props }">
      <v-btn
        v-bind="props"
        icon="mdi-delete"
        size="small"
        class="mr-2"
        color="error"
      />
    </template>
    <template #default="{ isActive }">
      <v-card title="Remove Entry?">
        <v-card-text>
          Remove {{ entry.name }}?
        </v-card-text>
        <v-card-actions>
          <v-spacer/>
          <v-btn
            text="Cancel"
            color="secondary"
            @click="isActive.value = false"
          />
          <v-btn
            text="Remove"
            color="primary"
            @click="emit('delete'); isActive.value = false"
          />
        </v-card-actions>
      </v-card>
    </template>
  </v-dialog>
  <v-text-field
    label="Name"
    v-model="entry.name"
    class="pr-1 flex-grow-1"
    density="compact"
    :rules="[validateName]"
    :persistent-hint="true"
  />
  <v-text-field
    label="# Entries"
    :model-value="entriesValue"
    class="pr-2"
    style="flex-grow: 0.25"
    @update:model-value="emit('set-entries', $event)"
    density="compact"
    :rules="[validateEntryCount]"
    :persistent-hint="true"
    :hint="props.rangeHint"
  />
  <v-btn
    icon="mdi-plus"
    class="mr-2"
    color="primary"
    @click="emit('add-entry')"
    size="small"
  />
  <v-btn
    icon="mdi-minus"
    color="primary"
    @click="emit('subtract-entry')"
    size="small"
  />
</template>

<script setup lang="ts">
import RaffleEntry from "@/types/RaffleEntry";
import {computed} from "vue";

const props = defineProps<{
  entry: RaffleEntry
  rangeHint: string
}>()

const emit = defineEmits<{
  (e: 'add-entry'): void,
  (e: 'subtract-entry'): void,
  (e: 'delete'): void,
  (e: 'set-entries', v: string): void,
}>()

const entriesValue = computed(() => {
  return props.entry.entries.toFixed(0)
})

const validateName = (value: string) => {
  if (!value || value.length === 0) {
    return 'May not be blank.'
  }
  return true
}

const validateEntryCount = (value: string) => {
  const val = parseInt(value)
  if (isNaN(val) || val < 1) {
    return 'Must be positive.'
  }
  return true
}


</script>

<style scoped>

</style>
