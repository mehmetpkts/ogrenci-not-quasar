<template>
  <q-layout view="lHh Lpr lFf">
    <q-header
      elevated
      class="bg-primary text-white"
    >
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg">
          </q-avatar>
          Öğrenci Bilgi Sistemi
        </q-toolbar-title>

        <q-space />

        <q-btn
          flat
          label="Hakkında"
          @click="hakkindaDialogOpen = true"
        />
        <q-btn
          flat
          label="İletişim"
          @click="iletisimDialogOpen = true"
        />

        <q-btn
          flat
          round
          dense
          icon="login"
          class="q-ml-md"
        />
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
    >
      <q-list>
        <q-item-label header>
          Menu
        </q-item-label>

        <q-item
          to="/"
          clickable
          v-ripple
          exact
        >
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Ana Sayfa</q-item-label>
            <q-item-label caption>
              Öğrenci Bilgi Sistemi
            </q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>

    <q-footer
      elevated
      class="bg-grey-8 text-white"
    >
      <q-toolbar>
        <q-toolbar-title>
          <div class="text-center">
            &copy; 2025 | Öğrenci Bilgi Sistemi - Tüm Hakları Saklıdır.
          </div>
        </q-toolbar-title>
      </q-toolbar>
    </q-footer>

    <HakkindaDialog v-model="hakkindaDialogOpen" />
    <IletisimDialog v-model="iletisimDialogOpen" />
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import HakkindaDialog from 'components/HakkindaDialog.vue'
import IletisimDialog from 'components/IletisimDialog.vue'

const leftDrawerOpen = ref(false)
const hakkindaDialogOpen = ref(false)
const iletisimDialogOpen = ref(false)

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value
}
</script>

<style lang="scss">
.q-toolbar__title {
  display: flex;
  align-items: center;
  gap: 10px;
}

.q-footer .q-toolbar__title {
  font-size: 0.8rem;
  text-align: center;
  width: 100%;
}
</style>