<template>
  <q-page class="q-pa-md">
    <div class="column q-gutter-md">
      <q-card>
        <q-card-section>
          <div class="text-h6">Öğrenciler</div>
        </q-card-section>
        <q-separator />
        <q-card-section>
          <pre class="q-ma-none">{{ j(ogrenciler) }}</pre>
        </q-card-section>
      </q-card>

      <q-card>
        <q-card-section>
          <div class="text-h6">Eğitmenler</div>
        </q-card-section>
        <q-separator />
        <q-card-section>
          <pre class="q-ma-none">{{ j(egitmenler) }}</pre>
        </q-card-section>
      </q-card>

      <q-card>
        <q-card-section>
          <div class="text-h6">Dersler</div>
        </q-card-section>
        <q-separator />
        <q-card-section>
          <pre class="q-ma-none">{{ j(dersler) }}</pre>
        </q-card-section>
      </q-card>

      <q-card>
        <q-card-section>
          <div class="text-h6">Notlar</div>
        </q-card-section>
        <q-separator />
        <q-card-section>
          <pre class="q-ma-none">{{ j(notlar) }}</pre>
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const ogrenciler = ref([])
const egitmenler = ref([])
const dersler = ref([])
const notlar = ref([])

const j = (v) => JSON.stringify(v, null, 2)

onMounted(() => {
  Promise.all([
    axios.get('http://localhost:8000/api/ogrenciler'),
    axios.get('http://localhost:8000/api/egitmenler'),
    axios.get('http://localhost:8000/api/dersler'),
    axios.get('http://localhost:8000/api/notlar')
  ]).then(([ogr, egt, drs, nt]) => {
    ogrenciler.value = ogr.data
    egitmenler.value = egt.data
    dersler.value = drs.data
    notlar.value = nt.data
  })
})
</script>
