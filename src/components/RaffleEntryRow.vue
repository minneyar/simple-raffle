<template>
  <EntryProperties
    :model-value="state.entry"
    @update:model-value="emit('update:model-value', $event)"
    @delete="emit('delete')"
  />
  <v-text-field
    label="Name"
    v-model="state.entry.name"
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
    @update:model-value="setEntries($event)"
    density="compact"
    :rules="[validateEntryCount]"
    :persistent-hint="true"
    :hint="props.rangeHint"
  />
  <v-btn
    icon="mdi-plus"
    class="mr-2"
    color="primary"
    @click="setEntries(state.entry.entries + 1)"
    size="small"
  />
  <v-btn
    icon="mdi-minus"
    color="primary"
    @click="setEntries(state.entry.entries - 1)"
    size="small"
  />
</template>

<script setup lang="ts">
import RaffleEntry from "@/types/RaffleEntry";
import {computed, onMounted, reactive, watch} from "vue";
import EntryProperties from "@/components/EntryProperties.vue";

const props = defineProps<{
  modelValue: RaffleEntry
  rangeHint: string
}>()

const state = reactive({
  entry: {name: '', entries: 0} as RaffleEntry
})

const emit = defineEmits<{
  (e: 'delete'): void,
  (e: 'update:model-value', v: RaffleEntry): void,
}>()

const entriesValue = computed(() => {
  return state.entry.entries.toFixed(0)
})

watch(() => props.modelValue, () => {
  state.entry = {
    name: props.modelValue.name,
    entries: props.modelValue.entries,
    contact: props.modelValue.contact,
  }
})

onMounted(() => {
  state.entry = {
    name: props.modelValue.name,
    entries: props.modelValue.entries,
    contact: props.modelValue.contact,
  }
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

const setEntries = (value: string | number) => {
  let val: number
  if (typeof (value) === 'number') {
    val = value
  } else {
    val = parseInt(value)
  }
  if (!isNaN(val)) {
    state.entry.entries = val
    emit('update:model-value', state.entry)
  }
}

</script>

<style scoped>

</style>
