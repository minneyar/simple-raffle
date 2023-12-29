<template>
  <v-dialog>
    <template #activator="{ props }">
      <v-btn
        v-bind="props"
        icon="mdi-dots-horizontal"
        size="small"
        class="mr-2"
        color="secondary"
      />
    </template>
    <template #default="{ isActive }">
      <v-form @submit.prevent="isActive.value = false; updateEntry()">
        <v-card :title="state.entry.name">
          <v-card-text>
            <v-text-field
              label="Contact Info"
              v-model="state.entry.contact"
              autofocus
            />
          </v-card-text>
          <v-card-actions>
            <DeleteButton
              :name="state.entry.name"
              @delete="emit('delete'); isActive.value = false"
            />
            <v-spacer/>
            <v-btn
              @click="isActive.value = false"
              color="secondary"
            >
              Cancel
            </v-btn>
            <v-btn
              color="primary"
              type="submit"
            >
              Update
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
    </template>
  </v-dialog>
</template>

<script setup lang="ts">
import RaffleEntry from "@/types/RaffleEntry";
import {onMounted, reactive, watch} from "vue";
import DeleteButton from "@/components/DeleteButton.vue";

const props = defineProps<{
  modelValue: RaffleEntry
}>()

const state = reactive({
  entry: {} as RaffleEntry
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

const emit = defineEmits<{
  (e: 'update:model-value', v: RaffleEntry): void
  (e: 'delete'): void
}>()

const updateEntry = () => {
  emit('update:model-value', state.entry)
}
</script>

<style scoped>

</style>
