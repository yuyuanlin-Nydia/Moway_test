<template>
  <q-dialog>
    <q-card class="" style="width: 500px">
      <q-card-section class="row">
        <q-space />
        <q-btn icon="close" flat round dense v-close-popup />
      </q-card-section>
      <q-card-section class="q-pt-none">
        <q-img :src="previewImg" alt="" class="previewImg" />
      </q-card-section>
    </q-card>
  </q-dialog>
</template>
<script setup>
const { previewImg } = defineProps({
  previewImg: {
    type: String,
    required: true,
  },
})
</script>
