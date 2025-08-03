<template>
  <q-page class="flex flex-center">
    <img
      alt="Quasar logo"
      src="~assets/quasar-logo-vertical.svg"
      style="width: 200px; height: 200px"
    />
    <q-btn
      flat
      color="primary"
      unelevated
      size="lg"
      icon="info"
      class="information-btn"
      @click="openFeedbackDialog"
    />
  </q-page>
</template>

<script setup>
import { useQuasar } from 'quasar'
import DialogFeedback from 'components/DialogFeedbackForm.vue'
const $q = useQuasar()
const openFeedbackDialog = () => {
  const feedbackDialog = $q
    .dialog({
      component: DialogFeedback,
    })
    .onOk(() => {
      feedbackDialog.hide()
    })
}
</script>

<style lang="scss" scoped>
.information-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
}
</style>
