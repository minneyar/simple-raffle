<template>
  <v-dialog>
    <template #activator="{ props }">
      <v-btn
        v-bind="props"
        prepend-icon="mdi-delete"
        class="mr-2"
        color="error"
      >
        Remove
      </v-btn>
    </template>
    <template #default="{ isActive }">
      <v-card title="Remove Entry?">
        <v-card-text>
          Remove {{ props.name }}?
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
</template>

<script setup lang="ts">
const props = defineProps<{
  name: string
}>()

const emit = defineEmits<{
  (e: 'delete'): void
}>()

</script>

<style scoped>

</style>
