<template>
  <div>
    <q-card-section class="row items-center q-pb-sm">
      <div class="text-h6">{{ title }}</div>
      <q-space />
      <q-btn color="primary" icon="add" label="Yeni Ekle" @click="openAddDialog" />
    </q-card-section>

    <q-card-section>
      <div class="q-mb-sm">
        <q-input v-model="filter" dense debounce="300" placeholder="Ara..." clearable filled>
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
        <div v-for="item in filteredRows" :key="item.id" class="col-12 col-sm-6 col-md-4">
          <q-card>
            <q-card-section>
              <div class="text-h6">{{ item.ad }}</div>
              <div v-if="getEgitmen(item.egitmen_id)" class="text-subtitle2 text-grey-7">{{ getEgitmen(item.egitmen_id) }}</div>
            </q-card-section>
            <q-separator />
            <q-card-actions align="right">
              <q-btn flat round color="warning" icon="edit" size="sm" @click="openEditDialog(item)" />
              <q-btn flat round color="negative" icon="delete" size="sm" @click="openDeleteDialog(item)" />
            </q-card-actions>
          </q-card>
        </div>
      </div>
    </q-card-section>

    <!-- Düzenle Dialog -->
    <q-dialog v-model="showEditDialog" persistent>
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <div class="text-h6">Dersi Düzenle</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section>
          <q-form @submit.prevent="onEditSave" class="q-gutter-md q-pa-md">
            <q-input v-model="editItem.ad" label="Ders Adı" filled dense :rules="[(v) => !!v || 'Ders Adı zorunlu']" />
            <q-select v-model="editItem.egitmen_id" :options="egitmenlerOptions" label="Eğitmen" filled dense emit-value map-options />
            <template v-for="k in extraKeys" :key="'edit-'+k">
              <q-input v-model="editItem[k]" :label="prettyLabel(k)" filled dense />
            </template>
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
          <div class="text-h6">Yeni Ders Ekle</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section>
          <q-form @submit="onSave" class="q-gutter-md q-pa-md">
            <q-input v-model="newItem.ad" label="Ders Adı" filled dense :rules="[(v) => !!v || 'Ders Adı zorunlu']" />
            <q-select v-model="newItem.egitmen_id" :options="egitmenlerOptions" label="Eğitmen" filled dense emit-value map-options />
            <template v-for="k in extraKeys" :key="'new-'+k">
              <q-input v-model="newItem[k]" :label="prettyLabel(k)" filled dense />
            </template>
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
        <q-card-section>Bu kaydı kalıcı olarak silmek istediğinizden emin misiniz?</q-card-section>
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
  egitmenler: { type: Array, default: () => [] },
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
  const base = { ad: '', egitmen_id: null }
  extraKeys.value.forEach(k => { base[k] = '' })
  newItem.value = base
  showAddDialog.value = true
}

const openDeleteDialog = (item) => {
  itemToDelete.value = item
  showDeleteDialog.value = true
}

const openEditDialog = (item) => {
  editItem.value = {
    id: item.id,
    ad: item.ad,
    egitmen_id: item.egitmen_id ?? item.egitmen?.id ?? null,
  }
  extraKeys.value.forEach(k => { editItem.value[k] = item[k] ?? '' })
  showEditDialog.value = true
}

const onSave = async () => {
  if (!newItem.value.ad) {
    $q.notify({ type: 'warning', message: 'Ders Adı zorunludur.' })
    return
  }

  try {
    saving.value = true
    await axios.post('http://localhost:8000/api/dersler', newItem.value)
    $q.notify({ type: 'positive', message: 'Ders başarıyla eklendi.' })
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
  const { id, ad } = editItem.value || {}
  if (!id || !ad) {
    $q.notify({ type: 'warning', message: 'Ders Adı zorunludur.' })
    return
  }
  try {
    savingEdit.value = true
    const payload = { ...editItem.value }
    delete payload.id
    await axios.put(`http://localhost:8000/api/dersler/${id}`, payload)
    $q.notify({ type: 'positive', message: 'Ders güncellendi.' })
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
    await axios.delete(`http://localhost:8000/api/dersler/${itemToDelete.value.id}`)
    showDeleteDialog.value = false
    emit('data-changed')
  } catch (error) {
    console.error("Silme hatası:", error)
    $q.notify({ type: 'negative', message: String(error?.response?.data?.message || error?.message || 'Silme hatası') })
  }
}

const isEmpty = computed(() => props.data.length === 0)

const rows = computed(() => props.data)

const filteredRows = computed(() => {
  if (!filter.value) {
    return rows.value
  }
  const lowerCaseFilter = filter.value.toLowerCase()
  return rows.value.filter(item => {
    const egitmenAdi = getEgitmen(item.egitmen_id) || ''
    return item.ad.toLowerCase().includes(lowerCaseFilter) || egitmenAdi.toLowerCase().includes(lowerCaseFilter)
  })
})

const hiddenKeys = ['id', 'created_at', 'updated_at', 'egitmen']
const baseFormKeys = ['ad', 'egitmen_id']
const extraKeys = computed(() => {
  const sample = props.data?.[0] || {}
  return Object.keys(sample).filter(k => !hiddenKeys.includes(k) && !baseFormKeys.includes(k))
})

const prettyLabel = (k) => {
  const map = { ad: 'Ders', egitmen_id: 'Eğitmen' }
  return map[k] || k.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase())
}

const egitmenlerOptions = computed(() =>
  props.egitmenler.map(e => ({ label: e.ad_soyad, value: e.id }))
)

const getEgitmen = (id) => {
    const egitmen = props.egitmenler.find(e => e.id === id)
    return egitmen ? egitmen.ad_soyad : ''
}

</script>
