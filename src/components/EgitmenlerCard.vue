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
              <div class="text-h6">{{ item.ad_soyad }}</div>
              <div class="text-subtitle2 text-grey-7">{{ item.egitmen_no }}</div>
            </q-card-section>
             <q-list dense padding>
                <q-item>
                    <q-item-section avatar>
                        <q-icon name="phone" />
                    </q-item-section>
                    <q-item-section>
                        {{ item.telefon_numarasi || item.telefon }}
                    </q-item-section>
                </q-item>
                <q-item v-if="item.email || item.okul_email || item.kurum_email">
                    <q-item-section avatar>
                        <q-icon name="email" />
                    </q-item-section>
                    <q-item-section>
                        {{ item.okul_email || item.kurum_email || item.email }}
                    </q-item-section>
                </q-item>
            </q-list>
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
          <div class="text-h6">Eğitmeni Düzenle</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section>
          <q-form @submit.prevent="onEditSave" class="q-gutter-md q-pa-md">
            <q-input v-model="editItem.ad_soyad" label="Adı Soyadı" filled dense :rules="[(v) => !!v || 'Adı Soyadı zorunlu']" />
            <q-input v-model="editItem.egitmen_no" label="Eğitmen No" filled dense :rules="[(v) => !!v || 'Eğitmen No zorunlu']" />
            <q-input v-model="editItem.telefon" label="Telefon Numarası" filled dense :rules="[(v) => !!v || 'Telefon Numarası zorunlu']" />
            <q-input v-model="editItem.email" label="E-posta" type="email" filled dense />
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
          <div class="text-h6">Yeni Eğitmen Ekle</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-card-section>
          <q-form @submit="onSave" class="q-gutter-md q-pa-md">
            <q-input v-model="newItem.ad_soyad" label="Adı Soyadı" filled dense :rules="[(v) => !!v || 'Adı Soyadı zorunlu']" />
            <q-input v-model="newItem.egitmen_no" label="Eğitmen No" filled dense :rules="[(v) => !!v || 'Eğitmen No zorunlu']" />
            <q-input v-model="newItem.telefon" label="Telefon Numarası" filled dense :rules="[(v) => !!v || 'Telefon Numarası zorunlu']" />
            <q-input v-model="newItem.email" label="E-posta" type="email" filled dense />
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
  newItem.value = { ad_soyad: '', egitmen_no: '', telefon: '', email: '' }
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
    egitmen_no: item.egitmen_no ?? '',
    telefon: item.telefon_numarasi ?? item.telefon ?? '',
    email: item.okul_email ?? item.kurum_email ?? item.email ?? '',
  }
  showEditDialog.value = true
}

const onSave = async () => {
  const { ad_soyad, egitmen_no, telefon, email } = newItem.value || {}
  if (!ad_soyad || !egitmen_no || !telefon) {
    $q.notify({ type: 'warning', message: 'Adı Soyadı, Eğitmen No ve Telefon zorunludur.' })
    return
  }

  try {
    saving.value = true
    const payload = { ad_soyad, egitmen_no, [phoneField.value]: telefon, [emailField.value]: email }
    await axios.post('http://localhost:8000/api/egitmenler', payload)
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
  const { id, ad_soyad, egitmen_no, telefon, email } = editItem.value || {}
  if (!id || !ad_soyad || !egitmen_no || !telefon) {
    $q.notify({ type: 'warning', message: 'Adı Soyadı, Eğitmen No ve Telefon zorunludur.' })
    return
  }
  try {
    savingEdit.value = true
    const payload = { ad_soyad, egitmen_no, [phoneField.value]: telefon, [emailField.value]: email }
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

const rows = computed(() => props.data)

const filteredRows = computed(() => {
  if (!filter.value) {
    return rows.value
  }
  const lowerCaseFilter = filter.value.toLowerCase()
  return rows.value.filter(item => {
    return Object.values(item).some(val => 
      String(val).toLowerCase().includes(lowerCaseFilter)
    )
  })
})

// Backend'de alan adlarının tutarlılığını yönetmek için
const phoneField = computed(() => {
  const sample = props.data?.[0] || {}
  return 'telefon_numarasi' in sample ? 'telefon_numarasi' : 'telefon'
})

const emailField = computed(() => {
  const sample = props.data?.[0] || {}
  if ('okul_email' in sample) return 'okul_email'
  if ('kurum_email' in sample) return 'kurum_email'
  return 'email'
})
</script>
