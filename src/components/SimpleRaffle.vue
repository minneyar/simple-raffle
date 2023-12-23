<template>
  <v-card
    class="mx-auto fill-height"
  >
    <v-layout>
      <v-app-bar
        color="primary"
        density="compact"
      >
        <v-app-bar-title>Raffle</v-app-bar-title>

        <v-spacer/>

        <v-btn
          prepend-icon="mdi-sort-alphabetical-ascending"
          @click="sortEntries"
          size="small"
        >
          Sort
        </v-btn>

        <v-dialog>
          <template #activator="{ props }">
            <v-btn
              v-bind="props"
              prepend-icon="mdi-party-popper"
              size="small"
            >
              Winner
            </v-btn>
          </template>
          <template #default="{ isActive }">
            <v-card>
              <v-card-title>Winner</v-card-title>
              <v-card-text>{{ pickWinner() }}</v-card-text>
              <v-card-actions>
                <v-spacer/>
                <v-btn
                  color="primary"
                  @click="isActive.value = false"
                >
                  Yay!
                </v-btn>
              </v-card-actions>
            </v-card>
          </template>
        </v-dialog>

        <v-dialog>
          <template #activator="{ props }">
            <v-btn
              v-bind="props"
              prepend-icon="mdi-plus"
              size="small"
            >
              Add
            </v-btn>
          </template>
          <template #default="{ isActive }">
            <v-form @submit.prevent>
              <v-card>
                <v-card-title>Add Name</v-card-title>
                <v-card-text>
                  <v-text-field
                    :autofocus="true"
                    label="Name"
                    v-model="state.newName"
                  />
                </v-card-text>
                <v-card-actions>
                  <v-spacer/>
                  <v-btn
                    color="primary"
                    type="submit"
                    @click="addEntry(); isActive.value = false"
                  >
                    Add Entry
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-form>
          </template>
        </v-dialog>
      </v-app-bar>
      <v-main class="fill-height">
        <v-container fluid class="pl-2 pr-2 pt-6">
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
                        @click="state.entries.splice(index, 1); isActive.value = false"
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
                :model-value="entry.entries.toFixed(0)"
                class="pr-2"
                style="flex-grow: 0.25"
                @update:model-value="updateEntries(index, $event)"
                density="compact"
                :rules="[validateEntryCount]"
                :persistent-hint="true"
                :hint="(entryRanges.length > index) ? entryRanges[index]: ''"
              />
              <v-btn
                icon="mdi-plus"
                class="mr-2"
                color="primary"
                @click="state.entries[index].entries += 1"
                size="small"
              />
              <v-btn
                icon="mdi-minus"
                color="primary"
                @click="state.entries[index].entries -= 1"
                size="small"
              />
            </v-col>
          </v-row>
        </v-container>
      </v-main>
    </v-layout>

  </v-card>
</template>

<script setup lang="ts">

import {computed, reactive, watch} from "vue";

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
  newName: '',
})

const addEntry = () => {
  state.entries.push({name: state.newName, entries: 1})
  state.newName = ''
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
  return `#${winner + 1} (${pickList[winner]}) wins!`
}

const validateName = (value: string) => {
  if (value.length === 0) {
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

const sortEntries = () => {
  state.entries.sort((a, b) => a.name.localeCompare(b.name))
}

const entryRanges = computed(() => {
  const ranges: string[] = []
  let startVal = 0
  let endVal = 0
  state.entries.forEach(entry => {
    endVal = startVal + entry.entries
    if (startVal + 1 === endVal) {
      ranges.push(`${endVal}`)
    } else {
      ranges.push(`${startVal + 1}-${endVal}`)
    }
    startVal = endVal
  })
  return ranges
})

watch(() => [state.entries], () => {
  localStorage.setItem("raffleEntries", JSON.stringify(state.entries))

}, {deep: true})

</script>

<style scoped>
.entry-row {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
}
</style>
