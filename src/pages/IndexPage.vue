<template>
  <q-page class="q-pa-md bg-grey-2">
    <div class="q-mx-auto" style="max-width: 1200px">
      <div class="row items-center q-mb-md">
        <q-icon name="school" size="lg" color="primary" />
        <h4 class="q-ma-none q-ml-md text-weight-bold text-primary">
          Öğrenci Bilgi Sistemi
        </h4>
      </div>

      <q-card class="q-mt-md">
        <q-tabs
          v-model="tab"
          class="text-grey"
          active-color="primary"
          indicator-color="primary"
          align="justify"
          narrow-indicator
        >
          <q-tab name="ogrenciler" label="Öğrenciler" icon="groups" />
          <q-tab name="egitmenler" label="Eğitmenler" icon="person" />
          <q-tab name="dersler" label="Dersler" icon="menu_book" />
          <q-tab name="notlar" label="Notlar" icon="grading" />
        </q-tabs>

        <q-separator />

        <q-tab-panels v-model="tab" animated>
          <q-tab-panel name="ogrenciler" class="q-pa-none">
            <OgrencilerCard id="ogrenciler" :data="ogrenciler" title="Öğrenciler" @data-changed="refreshData" />
          </q-tab-panel>

          <q-tab-panel name="egitmenler" class="q-pa-none">
            <EgitmenlerCard id="egitmenler" :data="egitmenler" title="Eğitmenler" @data-changed="refreshData" />
          </q-tab-panel>

          <q-tab-panel name="dersler" class="q-pa-none">
            <DerslerCard id="dersler" :data="dersler" :egitmenler="egitmenler" title="Dersler" @data-changed="refreshData" />
          </q-tab-panel>

          <q-tab-panel name="notlar" class="q-pa-none">
            <NotlarCard id="notlar" :data="notlar" :ogrenciler="ogrenciler" :dersler="dersler" title="Notlar" @data-changed="refreshData" />
          </q-tab-panel>
        </q-tab-panels>
      </q-card>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import OgrencilerCard from 'components/OgrencilerCard.vue'
import EgitmenlerCard from 'components/EgitmenlerCard.vue'
import DerslerCard from 'components/DerslerCard.vue'
import NotlarCard from 'components/NotlarCard.vue'

const tab = ref('ogrenciler')
const ogrenciler = ref([])
const egitmenler = ref([])
const dersler = ref([])
const notlar = ref([])

const refreshData = () => {
  Promise.all([
    axios.get('http://localhost:8000/api/ogrenciler'),
    axios.get('http://localhost:8000/api/egitmenler'),
    axios.get('http://localhost:8000/api/dersler'),
    axios.get('http://localhost:8000/api/notlar')
  ]).then(([ogr, egt, drs, nt]) => {
    const ogrArr = Array.isArray(ogr.data) ? ogr.data : (ogr?.data?.data || [])
    const egtArr = Array.isArray(egt.data) ? egt.data : (egt?.data?.data || [])
    const drsArr = Array.isArray(drs.data) ? drs.data : (drs?.data?.data || [])
    const ntArr  = Array.isArray(nt.data)  ? nt.data  : (nt?.data?.data  || [])

    const mapById = (arr) => arr.reduce((m, x) => { if (x && x.id != null) m[x.id] = x; return m }, {})
    const ogrMap = mapById(ogrArr)
    const egtMap = mapById(egtArr)
    const drsMap = mapById(drsArr)

    const drsEnriched = drsArr.map(d => ({
      ...d,
      egitmen: d && (d.egitmen_id != null) ? egtMap[d.egitmen_id] || null : null,
    }))

    const ntEnriched = ntArr.map(n => ({
      ...n,
      ogrenci: n && (n.ogrenci_id != null) ? ogrMap[n.ogrenci_id] || null : null,
      ders:    n && (n.ders_id != null)    ? drsMap[n.ders_id]    || null : null,
    }))

    ogrenciler.value = ogrArr
    egitmenler.value = egtArr
    dersler.value = drsEnriched
    notlar.value = ntEnriched
  }).catch(error => {
    console.error("Veri yenileme hatası:", error)
  })
}

onMounted(() => {
  refreshData()
})
</script>
