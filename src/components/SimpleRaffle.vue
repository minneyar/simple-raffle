<template>
  <v-card
    class="mx-auto fill-height"
  >
    <v-layout class="fill-height">
      <v-app-bar
        color="primary"
        density="compact"
      >
        <template v-slot:prepend>
          <v-app-bar-nav-icon @click.stop="state.drawer = !state.drawer"/>
        </template>

        <v-app-bar-title>Raffle</v-app-bar-title>

        <v-spacer/>

        <v-btn
          prepend-icon="mdi-sort-alphabetical-ascending"
          @click="sortEntries"
          size="small"
          class="d-none d-sm-flex"
        >
          Sort
        </v-btn>
        <v-btn
          icon="mdi-sort-alphabetical-ascending"
          @click="sortEntries"
          class="d-flex d-sm-none"
        />

        <v-dialog>
          <template #activator="{ props }">
            <v-btn
              v-bind="props"
              prepend-icon="mdi-party-popper"
              size="small"
              class="d-none d-sm-flex"
            >
              Winner
            </v-btn>
            <v-btn
              v-bind="props"
              icon="mdi-party-popper"
              class="d-flex d-sm-none"
            />
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
              class="d-none d-sm-flex"
            >
              Add
            </v-btn>
            <v-btn
              v-bind="props"
              icon="mdi-plus"
              class="d-flex d-sm-none"
            />
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
                    @click="createEntry(); isActive.value = false"
                  >
                    Add Entry
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-form>
          </template>
        </v-dialog>
      </v-app-bar>

      <!--suppress HtmlUnknownBooleanAttribute -->
      <v-navigation-drawer
        v-model="state.drawer"
        location="left"
        temporary
      >
        <!--suppress HtmlUnknownBooleanAttribute -->
        <v-list nav>
          <v-list-item
            prepend-icon="mdi-upload"
            title="Import from CSV"
            value="import"
            @click="selectFile"
          />
          <v-list-item
            prepend-icon="mdi-download"
            title="Export to CSV"
            value="export"
            @click="exportCsv"
          />
          <v-dialog>
            <template #activator="{props}">
              <v-list-item
                v-bind="props"
                prepend-icon="mdi-delete"
                title="Remove All"
                value="remove-all"
              />
            </template>
            <template #default="{isActive}">
              <v-card>
                <v-card-title>Remove All</v-card-title>
                <v-card-text>Remove all entries?</v-card-text>
                <v-card-actions>
                  <v-spacer/>
                  <v-btn
                    color="secondary"
                    @click="isActive.value = false"
                  >
                    Cancel
                  </v-btn>
                  <v-btn
                    color="primary"
                    @click="state.entries = []; isActive.value = false"
                  >
                    Remove All
                  </v-btn>
                </v-card-actions>
              </v-card>
            </template>
          </v-dialog>
        </v-list>
        <v-form
          ref="csvImportForm"
          @submit.prevent="handleSubmit"
        >
          <input
            id="csv-upload"
            type="file"
            hidden
            @change="submitForm"
          />
          <input
            id="submit-button"
            type="submit"
            hidden
          >
        </v-form>
      </v-navigation-drawer>

      <v-main class="fill-height">
        <!--suppress HtmlUnknownBooleanAttribute -->
        <v-container fluid class="pl-2 pr-2 pt-6">
          <v-row
            v-for="(entry, index) of state.entries"
            :key="index"
          >
            <v-col
              class="entry-row pt-0 pb-3"
            >
              <RaffleEntryRow
                :entry="entry"
                :range-hint="(entryRanges.length > index) ? entryRanges[index]: ''"
                @add-entry="addEntry(index)"
                @subtract-entry="subtractEntry(index)"
                @delete="state.entries.splice(index, 1)"
                @set-entries="updateEntries(index, $event)"
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
import RaffleEntry from "@/types/RaffleEntry";
import RaffleEntryRow from "@/components/RaffleEntryRow.vue";
import {SubmitEventPromise} from "vuetify";
import {VForm} from "vuetify/components/VForm";
import Papa from 'papaparse'

const savedEntriesStr = localStorage.getItem("raffleEntries")
let savedEntries: RaffleEntry[] = []
if (savedEntriesStr) {
  savedEntries = JSON.parse(savedEntriesStr)
}

const state = reactive({
  entries: savedEntries,
  deleteDialog: false,
  drawer: false,
  newName: '',
})

const createEntry = async () => {
  state.entries.push({name: state.newName, entries: 1})
  state.newName = ''
}

const updateEntries = async (index: number, entries: string) => {
  const newVal = parseInt(entries)
  if (!isNaN(newVal) && newVal > 0 && state.entries.length > index) {
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
  // console.info(`Picking from: ${pickList}`)

  const winner = Math.floor(Math.random() * pickList.length)
  return `#${winner + 1} (${pickList[winner]}) wins!`
}

const sortEntries = async () => {
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

const addEntry = async (index: number) => {
  if (state.entries.length > index) {
    state.entries[index].entries += 1
  }
}

const subtractEntry = async (index: number) => {
  if (state.entries.length > index && state.entries[index].entries > 1) {
    state.entries[index].entries -= 1
  }
}

const selectFile = async () => {
  console.info('Clicking button')
  document.getElementById('csv-upload')?.click()
}

const submitForm = async () => {
  document.getElementById('submit-button')?.click()
}

const handleSubmit = async (e: SubmitEventPromise) => {
  console.info(`handleSubmit: ${e}`)
  const el = document.getElementById('csv-upload')
  if (el instanceof HTMLInputElement) {
    if (el.files && el.files.length > 0) {
      const file = el.files.item(0)
      if (file) {
        Papa.parse<RaffleEntry>(file, {
          header: true,
          transform(value, field) {
            if (field === 'entries') {
              return parseInt(value)
            }
            return value
          },
          complete(results, _file) {
            if (results.errors.length === 0) {
              state.entries = results.data
            }
          },
        })
      }
    }
  }
}

const exportCsv = async () => {
  const data = Papa.unparse(state.entries, {header: true})

  const el = document.createElement('a')
  el.setAttribute('href', `data:text/plain;charset=utf-8,${encodeURIComponent(data)}`)
  el.setAttribute('download', 'raffle.csv')
  el.style.display = 'none'
  document.body.appendChild(el)
  el.click()
  document.body.removeChild(el)
}

watch(() => [state.entries], async () => {
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
