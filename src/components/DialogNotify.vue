<template>
  <q-dialog>
    <q-card class="dialogNotifyCard">
      <q-card-section class="row items-center q-pb-none">
        <q-icon color="green" :name="icon" size="md" class="q-mr-xs"></q-icon>
        <div class="text-h6 text-green">{{ title }}</div>
        <q-space />
        <q-btn icon="close" flat round dense v-close-popup />
      </q-card-section>

      <q-card-section>
        {{ content }}
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script setup>
const { title, content } = defineProps({
  icon: {
    type: String,
    required: false,
  },
  title: {
    type: String,
    required: true,
  },
  content: {
    type: String,
    required: true,
  },
})
</script>
<style lang="scss" scoped>
.dialogNotifyCard {
  max-width: 600px;
  width: 400px;
}
</style>
