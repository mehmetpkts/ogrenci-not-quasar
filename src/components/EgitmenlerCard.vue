<template>
  <div>
    <q-card-section>
      <!-- Boş durum -->
      <div v-if="isEmpty" class="text-center text-grey-6 q-pa-lg">
        <q-icon name="inbox" size="lg" />
        <div class="text-h6 q-mt-sm">Veri bulunamadı</div>
        <div class="text-caption">Bu bölüm için gösterilecek veri yok.</div>
      </div>

      <!-- Dizi + Nesne: Tablo -->
      <q-table
        v-else-if="isArrayOfObjects"
        flat
        :rows="rows"
        :columns="columns"
        row-key="__row_key"
        :pagination="{ rowsPerPage: 10 }"
      >
        <template #body-cell-actions="props">
          <q-td :props="props">
            <q-btn flat round color="primary" icon="visibility" size="sm" @click="openDetails(props.row)">
              <q-tooltip>Detayları Gör</q-tooltip>
            </q-btn>
          </q-td>
        </template>
      </q-table>

      <!-- Dizi + Primitive: Liste -->
      <q-list v-else-if="Array.isArray(data)" separator>
        <q-item v-for="(item, idx) in data" :key="idx">
          <q-item-section>{{ item }}</q-item-section>
          <q-item-section side>
            <q-btn flat round color="primary" icon="visibility" size="sm" @click="openDetails(item)">
              <q-tooltip>Detayları Gör</q-tooltip>
            </q-btn>
          </q-item-section>
        </q-item>
      </q-list>

      <!-- Tekil Nesne: Key-Value -->
      <q-list v-else separator>
        <q-item v-for="(val, key) in filteredObject(data)" :key="key">
          <q-item-section>
            <q-item-label overline class="text-grey-7">{{ key }}</q-item-label>
            <q-item-label>{{ val }}</q-item-label>
          </q-item-section>
        </q-item>
        <q-item>
          <q-item-section>
            <q-btn outline color="primary" label="Tüm Alanları Gör" @click="openDetails(data)" />
          </q-item-section>
        </q-item>
      </q-list>
    </q-card-section>

    <!-- Detay Dialog -->
    <q-dialog v-model="showDialog">
      <q-card style="min-width: 600px; max-width: 90vw">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">{{ title }} Detayları</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-separator />
        <q-card-section class="q-pt-md">
          <q-list bordered separator v-if="detailAsObject">
            <q-item v-for="(v, k) in selectedAllFields" :key="k">
              <q-item-section class="col-4 text-weight-medium text-grey-7">{{ k }}</q-item-section>
              <q-item-section>
                <pre class="q-ma-none" style="white-space: pre-wrap">{{ v }}</pre>
              </q-item-section>
            </q-item>
          </q-list>
          <div v-else>
            <pre class="q-ma-none" style="white-space: pre-wrap">{{ selectedAllFields }}</pre>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  data: {
    type: [Array, Object, String, Number, Boolean, null],
    default: () => [],
  },
  title: {
    type: String,
    default: 'Başlık'
  }
})

const showDialog = ref(false)
const selectedAllFields = ref(null)
const detailAsObject = computed(() => selectedAllFields.value && typeof selectedAllFields.value === 'object')

const openDetails = (row) => {
  selectedAllFields.value = row && row.__row_key !== undefined ? stripRowKey(row) : row
  showDialog.value = true
}

const stripRowKey = (r) => {
  if (r && typeof r === 'object') {
    const rest = { ...r }
    delete rest.__row_key
    return rest
  }
  return r
}

const ignoredKeys = ['id', 'created_at', 'updated_at', 'deleted_at']
const isIgnored = (k) => ignoredKeys.includes(k) || /_id$/.test(k)

const isEmpty = computed(() => {
  if (Array.isArray(props.data)) return props.data.length === 0
  if (props.data && typeof props.data === 'object') return Object.keys(props.data).length === 0
  return props.data === undefined || props.data === null || props.data === ''
})

const isArrayOfObjects = computed(() => {
  return Array.isArray(props.data) && props.data.length > 0 && typeof props.data[0] === 'object' && props.data[0] !== null
})

const rows = computed(() => {
  if (!isArrayOfObjects.value) return []
  return props.data.map((r, i) => ({ __row_key: i, ...r }))
})

const columns = computed(() => {
  if (!isArrayOfObjects.value) return []
  const keys = Object.keys(props.data[0]).filter((k) => !isIgnored(k))
  const base = keys.map((k) => ({ name: k, label: k, field: k, align: 'left', sortable: true }))
  return [
    ...base,
    { name: 'actions', label: 'İşlem', field: 'actions', align: 'right', sortable: false },
  ]
})

const filteredObject = (obj) => {
  if (!obj || typeof obj !== 'object') return obj
  const out = {}
  Object.keys(obj).forEach((k) => {
    if (!isIgnored(k)) out[k] = obj[k]
  })
  return out
}
</script>
