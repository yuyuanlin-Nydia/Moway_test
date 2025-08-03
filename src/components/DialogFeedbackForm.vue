<template>
  <q-dialog>
    <q-card class="dialogFeedbackCard">
      <q-card-section class="row items-center q-pb-none">
        <div class="text-h6">意見反饋</div>
        <q-space />
        <q-btn icon="close" flat round dense v-close-popup />
      </q-card-section>
      <q-form @submit="submit" greedy class="q-gutter-md">
        <q-card-section>
          <div class="q-mb-md">
            <div><span class="red q-mr-xs">*</span>類別</div>
            <q-select
              dense
              outlined
              v-model="form.type"
              :options="typeOptions"
              :rules="[(val) => !!val || '此欄位為必填']"
              placeholder="請選擇類別"
              option-value="value"
              option-label="label"
              map-options
              emit-value
            />
          </div>

          <div class="q-mb-md">
            <div><span class="red q-mr-xs">*</span>標題(限30字符內)</div>
            <q-input
              outlined
              v-model="form.title"
              dense
              :rules="[(val) => !!val || '此欄位為必填', (val) => val.length <= 30 || '限30字符內']"
              placeholder="請輸入標題"
            />
          </div>
          <div class="q-mb-md">
            <div><span class="red q-mr-xs">*</span>描述(限300字符內)</div>
            <q-input
              outlined
              v-model="form.description"
              dense
              autogrow
              :rules="[
                (val) => !!val || '此欄位為必填',
                (val) => val.length <= 300 || '限300字符內',
              ]"
              placeholder="請描述您的意見問題"
            />
          </div>
          <div class="q-mb-md">
            <div>參考圖片 (僅支援PNG、JPG格式，每個5MB內)</div>
            <input
              ref="fileInput"
              type="file"
              accept="image/png, image/jpeg"
              multiple
              class="hidden"
              @change="onFileChange"
            />
            <div class="row q-gutter-sm q-my-xs">
              <template v-if="filePreviewArr.length">
                <div v-for="(filePreview, index) in filePreviewArr" :key="index">
                  <div class="previewBox">
                    <img :src="filePreview" alt="" class="previewImg" />
                    <div class="iconBox">
                      <q-btn
                        flat
                        icon="zoom_in"
                        size="md"
                        color="white"
                        class="q-pa-xs"
                        @click="zoomInImg(index)"
                      />
                      <q-btn
                        flat
                        icon="delete"
                        size="md"
                        color="white"
                        class="q-pa-xs"
                        @click="deleteImg(index)"
                      />
                    </div>
                  </div>
                </div>
              </template>
              <div class="addImageBox" @click="triggerFileInput" v-if="!isOverImgLimit">
                <q-icon name="add" size="md" />
              </div>
            </div>

            <div class="text-red">{{ fileError }}</div>

            <div>
              可提供意見/問題截圖(上傳數量{{ isOverImgLimit ? '已達上限' : '' }}
              <span :class="{ 'text-green': hasSelectImage }">{{ filePreviewArr.length }} </span>/3)
              <br />
              <span v-if="isOverImgLimit" class="text-grey-6">
                請先刪除已上傳圖片，才能再上傳新圖片
              </span>
            </div>
          </div>
          <q-btn color="blue-1" class="text-blue-7 full-width" unelevated type="submit">提交</q-btn>
        </q-card-section>
      </q-form>
    </q-card>
  </q-dialog>
</template>

<script setup>
import { useQuasar } from 'quasar'
import { reactive, ref, computed, watch } from 'vue'
import DialogZoomInImage from './DialogZoomInImage.vue'
import DialogNotify from './DialogNotify.vue'

const maxSize = 5 * 1024 * 1024 // 5MB in bytes
const $q = useQuasar()
const form = reactive({
  type: '',
  title: '',
  description: '',
  imageArr: [],
})
const fileInput = ref(null)
const filePreviewArr = ref([])
const fileError = ref('')
const typeOptions = ref([
  { label: '操作問題', value: 'operation' },
  { label: '優化建議', value: 'optimize' },
  { label: 'Bug反饋', value: 'bug' },
  { label: '其他', value: 'other' },
])

const emit = defineEmits(['ok'])

const isOverImgLimit = computed(() => filePreviewArr.value.length >= 3)
const hasSelectImage = computed(() => filePreviewArr.value.length)
watch(
  () => form.imageArr,
  (val) => {
    fileError.value = ''
    val.forEach((file) => {
      if (file.size > maxSize) {
        fileError.value = '檔案大小不得超過 5MB'
      }
    })
  },
  { deep: true },
)

function triggerFileInput() {
  fileInput.value.click()
}

async function onFileChange(event) {
  const files = event.target.files
  if (!files.length) return

  resetFileError()
  filePreviewArr.value = []
  form.imageArr = Array.from(files)
  const images = []
  for (const [index, file] of form.imageArr.entries()) {
    //超過三個不顯示
    if (index > 2) {
      return
    }
    if (file.size > maxSize) {
      fileError.value = '檔案大小不得超過 5MB'
    }
    const base64Image = await handlePreview(file)
    images.push(base64Image)
  }
  filePreviewArr.value = images
}

function resetFileError() {
  if (fileError.value) {
    fileError.value = ''
  }
}

async function handlePreview(file) {
  var reader = new FileReader()
  return new Promise((resolve) => {
    reader.onload = (ev) => {
      resolve(ev.target.result)
    }
    reader.readAsDataURL(file)
  })
}

function deleteImg(index) {
  form.imageArr.splice(index, 1)
  filePreviewArr.value.splice(index, 1)
}

function zoomInImg(index) {
  $q.dialog({
    component: DialogZoomInImage,
    componentProps: {
      previewImg: filePreviewArr.value[index],
    },
  })
}
function submit() {
  if (fileError.value) {
    return
  }
  const dialog = $q.dialog({
    component: DialogNotify,
    componentProps: {
      icon: 'check_circle',
      title: '提交成功',
      content: '感謝您的努力，我們將繼續努力，提供最優質的服務!',
    },
  })
  let timer = setTimeout(() => {
    dialog.hide()
    emit('ok')
    clearTimeout(timer)
  }, 5000)
}
</script>
<style lang="scss" scoped>
.dialogFeedbackCard {
  max-width: 600px;
  width: 75vw;
  color: rgb(111, 111, 111);
  .addImageBox {
    cursor: pointer;
    width: 100px;
    height: 100px;
    border: 1px dashed lightgrey;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 8px;
  }
  .previewImg {
    width: 100%;
    height: 100%;
    border-radius: 8px;
    object-fit: cover;
  }
  .previewBox {
    position: relative;
    width: 100px;
    height: 100px;
    border-radius: 8px;
    .iconBox {
      visibility: hidden;
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 8px;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.7);
      padding: 0;
    }
    &:hover {
      .iconBox {
        visibility: visible;
      }
    }
  }
}
</style>
