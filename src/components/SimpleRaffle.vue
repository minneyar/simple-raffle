<template>
  <v-card
    class="mx-auto fill-height"
  >
    <v-layout>
      <v-app-bar
        color="primary"
        density="compact"
      >
        <template v-slot:prepend>
          <v-app-bar-nav-icon/>
        </template>
        <v-app-bar-title>Simple Raffle</v-app-bar-title>
      </v-app-bar>
      <v-main>
        <v-container fluid class="pt-6">
          <v-row
            v-for="(entry, index) of state.entries"
            :key="index"
          >
            <v-col
              class="entry-row pt-0 pb-3"
            >
              <v-dialog>
                <template #activator="{ props }">
                  <v-btn
                    v-bind="props"
                    icon="mdi-delete"
                    size="small"
                    class="mr-3"
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
                        @click="state.entries.splice(index, 1); isActive.value = false"
                      />
                    </v-card-actions>
                  </v-card>
                </template>
              </v-dialog>
              <v-text-field
                label="Name"
                v-model="entry.name"
                class="pr-3 flex-grow-1"
                :hide-details="true"
                density="compact"
              />
              <v-text-field
                label="# Entries"
                :model-value="entry.entries.toFixed(0)"
                class="pr-3"
                style="flex-grow: 0.25"
                @update:model-value="updateEntries(index, $event)"
                :hide-details="true"
                density="compact"
              />
              <v-btn
                icon="mdi-plus"
                class="mr-3"
                color="primary"
                @click="state.entries[index].entries += 1"
              />
              <v-btn
                icon="mdi-minus"
                color="primary"
                @click="state.entries[index].entries -= 1"
              />
            </v-col>
          </v-row>
        </v-container>
      </v-main>
    </v-layout>
    <v-sheet
      class="button-container">
      <v-dialog>
        <template #activator="{ props }">
          <v-btn
            v-bind="props"
            color="primary mr-3"
          >Pick Winner
          </v-btn>
        </template>
        <template #default="{ isActive }">
          <v-card>
            <v-card-title>Winner</v-card-title>
            <v-card-text>{{ pickWinner() }} wins!</v-card-text>
            <v-card-actions>
              <v-spacer/>
              <v-btn
                color="primary"
                @click="isActive.value = false"
              >Yay!
              </v-btn>
            </v-card-actions>
          </v-card>
        </template>
      </v-dialog>
      <v-btn
        icon="mdi-plus"
        color="primary"
        @click="addEntry"
      />
    </v-sheet>
  </v-card>
</template>

<script setup lang="ts">

import {reactive, watch} from "vue";

interface RaffleEntry {
  name: string
  entries: number
}

const savedEntriesStr = localStorage.getItem("raffleEntries")
let savedEntries: RaffleEntry[] = []
if (savedEntriesStr) {
  savedEntries = JSON.parse(savedEntriesStr)
}

const state = reactive({
  entries: savedEntries,
  deleteDialog: false,
})

const addEntry = () => {
  state.entries.push({name: "Name", entries: 5})
}

const updateEntries = (index: number, entries: string) => {
  const newVal = parseInt(entries)
  if (!isNaN(newVal) && newVal > 0) {
    state.entries[index].entries = newVal
  }
}

const pickWinner = () => {
  const pickList: string[] = []
  state.entries.forEach((entry) => {
    for (let i = 0; i < entry.entries; i++) {
      pickList.push(entry.name)
    }
  })
  console.info(`Picking from: ${pickList}`)

  const winner = Math.floor(Math.random() * pickList.length)
  return pickList[winner]
}

watch(() => [state.entries], () => {
  localStorage.setItem("raffleEntries", JSON.stringify(state.entries))
}, {deep: true})

</script>

<style scoped>
.entry-row {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.button-container {
  position: absolute;
  right: 5px;
  bottom: 5px;
  display: flex;
  align-items: center;
}
</style>
