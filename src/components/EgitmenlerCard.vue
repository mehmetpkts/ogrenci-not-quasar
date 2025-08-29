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
              color="warning"
              icon="edit"
              size="sm"
              @click="openEditDialog(props.row)"
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

    <!-- Düzenle Dialog -->
    <q-dialog
      v-model="showEditDialog"
      persistent
    >
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">Eğitmeni Düzenle</div>
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
            @submit.prevent="onEditSave"
            class="q-gutter-md q-pa-md"
          >
            <q-input
              v-model="editItem.ad_soyad"
              label="Adı Soyadı"
              filled
              dense
              :rules="[(v) => !!v || 'Adı Soyadı zorunlu']"
            />
            <div class="q-pt-md">
              <q-btn
                label="Güncelle"
                type="submit"
                color="primary"
                :loading="savingEdit"
                :disable="savingEdit"
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

    <!-- Ekle Dialog -->
    <q-dialog
      v-model="showAddDialog"
      persistent
    >
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">Yeni Eğitmen Ekle</div>
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
              v-model="newItem.ad_soyad"
              label="Adı Soyadı"
              filled
              dense
              :rules="[(v) => !!v || 'Adı Soyadı zorunlu']"
            />
            <div class="q-pt-md">
              <q-btn
                label="Kaydet"
                type="submit"
                color="primary"
                :loading="saving"
                :disable="saving"
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
import { useQuasar } from 'quasar'

const props = defineProps({
  data: { type: Array, default: () => [] },
  title: { type: String, default: 'Başlık' },
})

const emit = defineEmits(['data-changed'])
const $q = useQuasar()

const showDetailsDialog = ref(false)
const showAddDialog = ref(false)
const showDeleteDialog = ref(false)
const showEditDialog = ref(false)

const selectedItem = ref(null)
const newItem = ref({})
const filter = ref('')
const saving = ref(false)
const itemToDelete = ref(null)
const editItem = ref({})
const savingEdit = ref(false)

const openDetails = (item) => {
  selectedItem.value = item
  showDetailsDialog.value = true
}

const openAddDialog = () => {
  newItem.value = { ad_soyad: '' }
  showAddDialog.value = true
}

const openDeleteDialog = (item) => {
  itemToDelete.value = item
  showDeleteDialog.value = true
}

const openEditDialog = (item) => {
  editItem.value = {
    id: item.id,
    ad_soyad: item.ad_soyad,
  }
  showEditDialog.value = true
}

const onSave = async () => {
  if (!newItem.value.ad_soyad) {
    $q.notify({ type: 'warning', message: 'Adı Soyadı zorunludur.' })
    return
  }

  try {
    saving.value = true
    await axios.post('http://localhost:8000/api/egitmenler', newItem.value)
    $q.notify({ type: 'positive', message: 'Eğitmen başarıyla eklendi.' })
    showAddDialog.value = false
    emit('data-changed')
  } catch (error) {
    const msg = error?.response?.data?.message || error?.response?.data || error?.message || 'Kaydetme hatası'
    console.error('Kaydetme hatası:', error?.response ? error.response.data : error)
    $q.notify({ type: 'negative', message: String(msg) })
  } finally {
    saving.value = false
  }
}

const onEditSave = async () => {
  const { id, ad_soyad } = editItem.value || {}
  if (!id || !ad_soyad) {
    $q.notify({ type: 'warning', message: 'Adı Soyadı zorunludur.' })
    return
  }
  try {
    savingEdit.value = true
    const payload = { ad_soyad }
    await axios.put(`http://localhost:8000/api/egitmenler/${id}`, payload)
    $q.notify({ type: 'positive', message: 'Eğitmen güncellendi.' })
    showEditDialog.value = false
    emit('data-changed')
  } catch (error) {
    const msg = error?.response?.data?.message || error?.response?.data || error?.message || 'Güncelleme hatası'
    console.error('Güncelleme hatası:', error?.response ? error.response.data : error)
    $q.notify({ type: 'negative', message: String(msg) })
  } finally {
    savingEdit.value = false
  }
}

const onDeleteConfirm = async () => {
  if (!itemToDelete.value) return
  try {
    await axios.delete(`http://localhost:8000/api/egitmenler/${itemToDelete.value.id}`)
    showDeleteDialog.value = false
    emit('data-changed')
  } catch (error) {
    console.error("Silme hatası:", error)
    $q.notify({ type: 'negative', message: String(error?.response?.data?.message || error?.message || 'Silme hatası') })
  }
}

const isEmpty = computed(() => props.data.length === 0)
const isArrayOfObjects = computed(() => props.data.length > 0 && typeof props.data[0] === 'object')

const rows = computed(() => props.data)

const columns = computed(() => {
  if (!isArrayOfObjects.value) return []
  const hidden = ['id', 'created_at', 'updated_at']
  const keys = Object.keys(props.data[0] || {}).filter(k => !hidden.includes(k))
  const labelMap = {
    ad_soyad: 'Ad Soyad',
    ad: 'Ad',
    soyad: 'Soyad',
    created_at: 'Oluşturulma',
    updated_at: 'Güncellenme',
  }
  const pretty = (k) => labelMap[k] || k.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase())
  const base = keys.map((k) => ({ name: k, label: pretty(k), field: k, align: 'left', sortable: true }))
  return [
    ...base,
    { name: 'actions', label: 'İşlemler', field: 'actions', align: 'right', sortable: false },
  ]
})

const detailEntries = computed(() => {
  const item = selectedItem.value || {}
  const hide = ['id', 'created_at', 'updated_at']
  return Object.entries(item).filter(([k]) => !hide.includes(k))
})
</script>
