<template>
  <div>
    <q-card-section class="row items-center q-pb-sm">
      <div class="text-h6">{{ title }}</div>
      <q-space />
      <q-btn color="primary" icon="add" label="Yeni Ekle" @click="openAddDialog" />
    </q-card-section>

    <q-card-section>
      <div v-if="isEmpty" class="text-center text-grey-6 q-pa-lg">
        <q-icon name="inbox" size="lg" />
        <div class="text-h6 q-mt-sm">Veri bulunamadı</div>
      </div>

      <q-table
        v-else-if="isArrayOfObjects"
        flat
        :rows="rows"
        :columns="columns"
        row-key="id"
      >
        <template #body-cell-actions="props">
          <q-td :props="props">
            <q-btn flat round color="primary" icon="visibility" size="sm" @click="openDetails(props.row)" />
            <q-btn flat round color="warning" icon="edit" size="sm" @click="openEditDialog(props.row)" />
            <q-btn flat round color="negative" icon="delete" size="sm" @click="openDeleteDialog(props.row)" />
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
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section class="q-pt-md">
          <q-list bordered separator v-if="selectedItem">
            <q-item v-for="(val, key) in selectedItem" :key="key">
              <q-item-section class="col-4 text-weight-medium text-grey-7">{{ key }}</q-item-section>
              <q-item-section>
                <pre class="q-ma-none" style="white-space: pre-wrap">{{ val }}</pre>
              </q-item-section>
            </q-item>
          </q-list>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Düzenle Dialog -->
    <q-dialog v-model="showEditDialog" persistent>
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">Notu Düzenle</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section>
          <q-form @submit.prevent="onEditSave" class="q-gutter-md q-pa-md">
            <q-select
              v-model="editItem.ogrenci_id"
              :options="ogrencilerOptions"
              label="Öğrenci"
              filled
              dense
              emit-value
              map-options
              :rules="[(v) => !!v || 'Öğrenci zorunlu']"
            />
            <q-select
              v-model="editItem.ders_id"
              :options="derslerOptions"
              label="Ders"
              filled
              dense
              emit-value
              map-options
              :rules="[(v) => !!v || 'Ders zorunlu']"
            />
            <q-input
              v-model.number="editItem.not"
              type="number"
              label="Puan"
              filled
              dense
              :rules="[(v) => v === 0 || !!v || 'Puan zorunlu', (v) => !Number.isNaN(Number(v)) || 'Geçerli bir sayı girin']"
            />
            <div class="q-pt-md">
              <q-btn label="Güncelle" type="submit" color="primary" :loading="savingEdit" :disable="savingEdit" />
              <q-btn label="İptal" flat class="q-ml-sm" v-close-popup />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Ekle Dialog -->
    <q-dialog v-model="showAddDialog" persistent>
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">Yeni Not Ekle</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section>
          <q-form @submit.prevent="onSave" class="q-gutter-md q-pa-md">
            <q-select
              v-model="newItem.ogrenci_id"
              :options="ogrencilerOptions"
              label="Öğrenci"
              filled
              dense
              emit-value
              map-options
              :rules="[(v) => !!v || 'Öğrenci zorunlu']"
            />
            <q-select
              v-model="newItem.ders_id"
              :options="derslerOptions"
              label="Ders"
              filled
              dense
              emit-value
              map-options
              :rules="[(v) => !!v || 'Ders zorunlu']"
            />
            <q-input
              v-model.number="newItem.not"
              type="number"
              label="Puan"
              filled
              dense
              :rules="[(v) => v === 0 || !!v || 'Puan zorunlu', (v) => !Number.isNaN(Number(v)) || 'Geçerli bir sayı girin']"
            />
            <div class="q-pt-md">
              <q-btn label="Kaydet" type="submit" color="primary" :loading="saving" :disable="saving" />
              <q-btn label="İptal" flat class="q-ml-sm" v-close-popup />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Silme Onay Dialog -->
    <q-dialog v-model="showDeleteDialog" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="warning" color="negative" text-color="white" />
          <span class="q-ml-sm text-h6">Silme Onayı</span>
        </q-card-section>
        <q-card-section>
          Bu kaydı kalıcı olarak silmek istediğinizden emin misiniz?
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="İptal" color="primary" v-close-popup />
          <q-btn label="Evet, Sil" color="negative" @click="onDeleteConfirm" />
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
  ogrenciler: { type: Array, default: () => [] },
  dersler: { type: Array, default: () => [] },
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
const saving = ref(false)
const itemToDelete = ref(null)
const editItem = ref({})
const savingEdit = ref(false)

const openDetails = (item) => {
  selectedItem.value = item
  showDetailsDialog.value = true
}

const openAddDialog = () => {
  newItem.value = { ogrenci_id: null, ders_id: null, not: null }
  showAddDialog.value = true
}

const openDeleteDialog = (item) => {
  itemToDelete.value = item
  showDeleteDialog.value = true
}

const openEditDialog = (item) => {
  // Mevcut kaydı form alanlarına koy
  editItem.value = {
    id: item.id,
    ogrenci_id: item.ogrenci_id ?? item.ogrenci?.id ?? null,
    ders_id: item.ders_id ?? item.ders?.id ?? null,
    not: item.not ?? item.puan ?? null,
  }
  showEditDialog.value = true
}

const onSave = async () => {
  // Basit doğrulama
  const { ogrenci_id, ders_id, not } = newItem.value || {}
  const ogrIdNum = Number(ogrenci_id)
  const drsIdNum = Number(ders_id)
  const notNum = Number(not)
  if (!ogrIdNum || !drsIdNum || Number.isNaN(notNum)) {
    $q.notify({ type: 'warning', message: 'Öğrenci, Ders ve geçerli bir Puan zorunludur.' })
    return
  }

  try {
    saving.value = true
    const payload = {
      ogrenci_id: ogrIdNum,
      ders_id: drsIdNum,
      not: notNum,
    }
    await axios.post('http://localhost:8000/api/notlar', payload)
    $q.notify({ type: 'positive', message: 'Not başarıyla eklendi.' })
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
  const { id, ogrenci_id, ders_id, not } = editItem.value || {}
  const ogrIdNum = Number(ogrenci_id)
  const drsIdNum = Number(ders_id)
  const notNum = Number(not)
  if (!id || !ogrIdNum || !drsIdNum || Number.isNaN(notNum)) {
    $q.notify({ type: 'warning', message: 'Öğrenci, Ders ve geçerli bir Puan zorunludur.' })
    return
  }
  try {
    savingEdit.value = true
    const payload = { ogrenci_id: ogrIdNum, ders_id: drsIdNum, not: notNum }
    await axios.put(`http://localhost:8000/api/notlar/${id}`, payload)
    $q.notify({ type: 'positive', message: 'Not güncellendi.' })
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
    await axios.delete(`http://localhost:8000/api/notlar/${itemToDelete.value.id}`)
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
  const hidden = ['id', 'created_at', 'updated_at', 'ogrenci', 'ders']
  const keys = Object.keys(props.data[0] || {}).filter(k => !hidden.includes(k))
  const base = keys.map((k) => ({ name: k, label: k, field: k, align: 'left', sortable: true, format: (val, row) => formatCell(val, row, k) }))
  return [
    ...base,
    { name: 'actions', label: 'İşlemler', field: 'actions', align: 'right', sortable: false },
  ]
})

const ogrencilerOptions = computed(() => 
  props.ogrenciler.map(o => ({ label: o.ad_soyad, value: o.id }))
)

const derslerOptions = computed(() => 
  props.dersler.map(d => ({ label: d.ad, value: d.id }))
)

const formatCell = (val, row, key) => {
  if (key === 'ogrenci_id' && row.ogrenci) {
    return `${row.ogrenci.ad_soyad} (ID: ${val})`
  }
  if (key === 'ders_id' && row.ders) {
    return `${row.ders.ad} (ID: ${val})`
  }
  if (val == null) return ''
  if (typeof val === 'object') {
    const displayKey = ['ad_soyad', 'ad', 'isim', 'name', 'title'].find((k) => k in val)
    if (displayKey) return String(val[displayKey])
    return JSON.stringify(val)
  }
  return String(val)
}
</script>
