<template>
  <q-dialog
    :model-value="modelValue"
    @update:model-value="(value) => $emit('update:modelValue', value)"
  >
    <q-card>
      <q-card-section class="row items-center q-pb-none">
        <div class="text-h6">Hakkında</div>
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
        <p>Bu proje, öğrencilerin, eğitmenlerin, derslerin ve notların yönetimi için bir bilgi sistemi olarak tasarlanmıştır.</p>
        <p>Quasar Framework ve Vue.js kullanılarak geliştirilmiştir.</p>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>
