<template>
  <div>
    <q-card-section class="row items-center q-pb-sm">
      <div class="text-h6">{{ title }}</div>
      <q-space />
      <q-btn
        color="primary"
        icon="add"
        label="Yeni Ekle"
        @click="openAddDialog"
      />
    </q-card-section>

    <q-card-section>
      <div
        class="q-mb-sm"
        v-if="isArrayOfObjects"
      >
        <q-input
          v-model="filter"
          dense
          debounce="300"
          placeholder="Ara"
          clearable
          filled
        >
          <template #append>
            <q-icon name="search" />
          </template>
        </q-input>
      </div>
      <div
        v-if="isEmpty"
        class="text-center text-grey-6 q-pa-lg"
      >
        <q-icon
          name="inbox"
          size="lg"
        />
        <div class="text-h6 q-mt-sm">Veri bulunamadı</div>
      </div>

      <q-table
        v-else-if="isArrayOfObjects"
        flat
        :rows="rows"
        :columns="columns"
        :filter="filter"
        row-key="id"
      >
        <template #body-cell-actions="props">
          <q-td :props="props">
            <q-btn
              flat
              round
              color="primary"
              icon="visibility"
              size="sm"
              @click="openDetails(props.row)"
            />
            <q-btn
              flat
              round
              color="negative"
              icon="delete"
              size="sm"
              @click="openDeleteDialog(props.row)"
            />
          </q-td>
        </template>
      </q-table>
    </q-card-section>

    <!-- Detay Dialog -->
    <q-dialog v-model="showDetailsDialog">
      <q-card style="min-width: 600px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">{{ title }} Detayları</div>
          <q-space />
          <q-btn
            icon="close"
            flat
            round
            dense
            v-close-popup
          />
        </q-card-section>
        <q-card-section class="q-pt-md">
          <q-list
            bordered
            separator
            v-if="selectedItem"
          >
            <q-item
              v-for="(entry, idx) in detailEntries"
              :key="idx"
            >
              <q-item-section class="col-4 text-weight-medium text-grey-7">{{ entry[0] }}</q-item-section>
              <q-item-section>
                <pre
                  class="q-ma-none"
                  style="white-space: pre-wrap"
                >{{ entry[1] }}</pre>
              </q-item-section>
            </q-item>
          </q-list>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Ekle Dialog -->
    <q-dialog
      v-model="showAddDialog"
      persistent
    >
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">Yeni Ders Ekle</div>
          <q-space />
          <q-btn
            icon="close"
            flat
            round
            dense
            v-close-popup
          />
        </q-card-section>
        <q-card-section>
          <q-form
            @submit="onSave"
            class="q-gutter-md q-pa-md"
          >
            <q-input
              v-model="newItem.ad"
              label="Ders Adı"
              filled
              dense
            />
            <q-select
              v-model="newItem.egitmen_id"
              :options="egitmenlerOptions"
              label="Eğitmen"
              filled
              dense
              emit-value
              map-options
            />
            <div class="q-pt-md">
              <q-btn
                label="Kaydet"
                type="submit"
                color="primary"
              />
              <q-btn
                label="İptal"
                flat
                class="q-ml-sm"
                v-close-popup
              />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Silme Onay Dialog -->
    <q-dialog
      v-model="showDeleteDialog"
      persistent
    >
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar
            icon="warning"
            color="negative"
            text-color="white"
          />
          <span class="q-ml-sm text-h6">Silme Onayı</span>
        </q-card-section>
        <q-card-section>
          Bu kaydı kalıcı olarak silmek istediğinizden emin misiniz?
        </q-card-section>
        <q-card-actions align="right">
          <q-btn
            flat
            label="İptal"
            color="primary"
            v-close-popup
          />
          <q-btn
            label="Evet, Sil"
            color="negative"
            @click="onDeleteConfirm"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const props = defineProps({
  data: { type: Array, default: () => [] },
  egitmenler: { type: Array, default: () => [] },
  title: { type: String, default: 'Başlık' },
})

const emit = defineEmits(['data-changed'])

const showDetailsDialog = ref(false)
const showAddDialog = ref(false)
const showDeleteDialog = ref(false)

const selectedItem = ref(null)
const newItem = ref({})
const filter = ref('')
const itemToDelete = ref(null)

const openDetails = (item) => {
  selectedItem.value = item
  showDetailsDialog.value = true
}

const openAddDialog = () => {
  newItem.value = { ad: '', egitmen_id: null }
  showAddDialog.value = true
}

const openDeleteDialog = (item) => {
  itemToDelete.value = item
  showDeleteDialog.value = true
}

const onSave = async () => {
  try {
    await axios.post('http://localhost:8000/api/dersler', newItem.value)
    showAddDialog.value = false
    emit('data-changed')
  } catch (error) {
    console.error("Kaydetme hatası:", error)
  }
}

const onDeleteConfirm = async () => {
  if (!itemToDelete.value) return
  try {
    await axios.delete(`http://localhost:8000/api/dersler/${itemToDelete.value.id}`)
    showDeleteDialog.value = false
    emit('data-changed')
  } catch (error) {
    console.error("Silme hatası:", error)
  }
}

const isEmpty = computed(() => props.data.length === 0)
const isArrayOfObjects = computed(() => props.data.length > 0 && typeof props.data[0] === 'object')

const rows = computed(() => props.data)

const columns = computed(() => {
  if (!isArrayOfObjects.value) return []
  const hidden = ['id', 'created_at', 'updated_at', 'egitmen']
  const keys = Object.keys(props.data[0] || {}).filter(k => !hidden.includes(k))
  const labelMap = {
    ad: 'Ders',
    egitmen_id: 'Eğitmen',
    created_at: 'Oluşturulma',
    updated_at: 'Güncellenme',
  }
  const pretty = (k) => labelMap[k] || k.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase())
  const base = keys.map((k) => ({ name: k, label: pretty(k), field: k, align: 'left', sortable: true, format: (val, row) => formatCell(val, row, k) }))
  return [
    ...base,
    { name: 'actions', label: 'İşlemler', field: 'actions', align: 'right', sortable: false },
  ]
})

const egitmenlerOptions = computed(() =>
  props.egitmenler.map(e => ({ label: e.ad_soyad, value: e.id }))
)

const formatCell = (val, row, key) => {
  if (key === 'egitmen_id') {
    if (row.egitmen && row.egitmen.ad_soyad) return String(row.egitmen.ad_soyad)
    return val == null ? '' : String(val)
  }
  if (val == null) return ''
  if (typeof val === 'object') {
    const displayKey = ['ad_soyad', 'ad', 'isim', 'name', 'title'].find((k) => k in val)
    if (displayKey) return String(val[displayKey])
    return JSON.stringify(val)
  }
  return String(val)
}

const detailEntries = computed(() => {
  const item = selectedItem.value || {}
  const hide = ['id', 'created_at', 'updated_at']

  return Object.entries(item)
    .filter(([k]) => !hide.includes(k))
    .map(([key, value]) => {
      // Eğer değer string ise ve JSON formatındaysa parse et
      let val = value
      try {
        if (typeof value === 'string' && (value.startsWith('{') || value.startsWith('['))) {
          val = JSON.parse(value)
        }
      } catch (error) {
        console.debug('JSON parse hatası:', error)
      }

      // Eğer değer bir obje ise ad_soyad veya name gibi bir alanı al
      if (val && typeof val === 'object' && !Array.isArray(val)) {
        const displayKey = ['ad_soyad', 'name', 'ad', 'title'].find(k => k in val)
        if (displayKey) {
          val = val[displayKey]
        }
      }

      return [key, val]
    })
})
</script>
