<template>
  <div>
    <q-card-section class="row items-center q-pb-sm">
      <div class="text-h6">{{ title }}</div>
      <q-space />
      <q-btn color="primary" icon="add" label="Yeni Ekle" @click="openAddDialog" />
    </q-card-section>

    <q-card-section>
      <div class="q-mb-sm">
        <q-input v-model="filter" dense debounce="300" placeholder="Öğrenci veya derse göre ara..." clearable filled>
          <template #append>
            <q-icon name="search" />
          </template>
        </q-input>
      </div>
      <div v-if="isEmpty" class="text-center text-grey-6 q-pa-lg">
        <q-icon name="inbox" size="lg" />
        <div class="text-h6 q-mt-sm">Veri bulunamadı</div>
      </div>

      <div v-else class="row q-col-gutter-md q-mt-md">
        <div v-for="ogrenci in filteredGroupedNotes" :key="ogrenci.ogrenci_id" class="col-12 col-sm-6 col-md-4">
          <q-card>
            <q-card-section class="bg-grey-2">
              <div class="text-h6">{{ ogrenci.ogrenci_ad_soyad }}</div>
            </q-card-section>

            <q-list separator>
              <q-item v-for="not in ogrenci.notlar" :key="not.id">
                <q-item-section>
                  <q-item-label>{{ not.ders_ad }}</q-item-label>
                </q-item-section>
                <q-item-section side>
                  <q-item-label caption>{{ not.not }}</q-item-label>
                </q-item-section>
                <q-item-section side>
                   <q-btn flat round color="warning" icon="edit" size="sm" @click="openEditDialog(not)" />
                   <q-btn flat round color="negative" icon="delete" size="sm" @click="openDeleteDialog(not)" />
                </q-item-section>
              </q-item>
            </q-list>
          </q-card>
        </div>
      </div>
    </q-card-section>

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

const showAddDialog = ref(false)
const showDeleteDialog = ref(false)
const showEditDialog = ref(false)

const newItem = ref({})
const filter = ref('')
const saving = ref(false)
const itemToDelete = ref(null)
const editItem = ref({})
const savingEdit = ref(false)

const openAddDialog = () => {
  newItem.value = { ogrenci_id: null, ders_id: null, not: null }
  showAddDialog.value = true
}

const openDeleteDialog = (item) => {
  itemToDelete.value = item
  showDeleteDialog.value = true
}

const openEditDialog = (item) => {
  editItem.value = { ...item }
  showEditDialog.value = true
}

const onSave = async () => {
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

const groupedNotes = computed(() => {
  const gruplar = new Map()

  props.data.forEach(not => {
    const ogrenciId = not.ogrenci_id
    if (!gruplar.has(ogrenciId)) {
      gruplar.set(ogrenciId, {
        ogrenci_id: ogrenciId,
        ogrenci_ad_soyad: not.ogrenci.ad_soyad,
        notlar: []
      })
    }
    gruplar.get(ogrenciId).notlar.push({
      id: not.id,
      ders_id: not.ders_id,
      ders_ad: not.ders.ad,
      not: not.not,
      ogrenci_id: not.ogrenci_id
    })
  })

  return Array.from(gruplar.values())
})

const filteredGroupedNotes = computed(() => {
  if (!filter.value) {
    return groupedNotes.value
  }
  const lowerCaseFilter = filter.value.toLowerCase()
  return groupedNotes.value.filter(ogrenci => {
    const adMatch = ogrenci.ogrenci_ad_soyad.toLowerCase().includes(lowerCaseFilter)
    const dersMatch = ogrenci.notlar.some(not => not.ders_ad.toLowerCase().includes(lowerCaseFilter))
    return adMatch || dersMatch
  })
})

const ogrencilerOptions = computed(() => 
  props.ogrenciler.map(o => ({ label: o.ad_soyad, value: o.id }))
)

const derslerOptions = computed(() => 
  props.dersler.map(d => ({ label: d.ad, value: d.id }))
)
</script>
