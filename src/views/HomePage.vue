<template>
  <div class="home-page">
    <AppHeader title="ULTRA Форматирование" />
    <div class="home-page__content">
      <n-spin v-if="fetchStatus === 'loading'" size="large" style="width: 100%;" />
      <template v-if="fetchStatus === 'init'">
        <BaseInstruction style="margin-bottom: 2rem" />
        <n-card style="margin-bottom: 1rem">
          <n-checkbox v-model:checked="checkboxes.grammar">
            Исправление грамматических ошибок
          </n-checkbox>
          <n-checkbox v-model:checked="checkboxes.html_formatting">
            Форматирование текста
          </n-checkbox>
        </n-card>
        <n-tabs type="segment" animated>
          <n-tab-pane  name="oasis" tab="Ввод текста">
            <n-input v-model:value="text" type="textarea" placeholder="Введите текст" style="min-height: 250px" />
            <n-button type="primary" @click="sendText" :disabled="!text" style="width: 100%; margin-top: 1rem;">Отправить</n-button>
          </n-tab-pane>
          <n-tab-pane name="the beatles" tab="Загрузка файла">
            <n-upload
                directory-dnd
                :on-change="downloadFile"
            >
              <n-upload-dragger>
                <div style="margin-bottom: 12px">
                  <n-icon size="48" :depth="3" :component="ArchiveOutline" />
                </div>
                <n-text style="font-size: 16px">
                  Кликните или перетащите файл
                </n-text>
                <n-p depth="3" style="margin: 8px 0 0 0">
                  Доступный формат файла: .txt, .docx
                </n-p>
              </n-upload-dragger>
            </n-upload>
          </n-tab-pane>
        </n-tabs>
      </template>
      <template v-if="fetchStatus === 'success' && result">
        <TextEditorInstruction style="margin-bottom: 2rem" />
        <TextEditor v-model:html="result.html" />
        <n-space justify="center" style="margin-top: 2rem">
          <n-button @click="sendNewRequest" type="primary" style="width: 300px; border-radius: 0.5rem;" >Отправить новый запрос</n-button>
        </n-space>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import {ref, watch} from "vue";
import { ArchiveOutline } from '@vicons/ionicons5'
import { useNotification } from 'naive-ui'
import type { UploadFileInfo } from 'naive-ui'
import AppHeader from "@/components/UI/AppHeader.vue";
import TextEditor from "@/components/TextEditor.vue";
import BaseInstruction from "@/components/BaseInstruction.vue";
import TextEditorInstruction from "@/components/TextEditorInstruction.vue";


interface IResult {
  html: string
}

const notification = useNotification()

const text = ref('')
const checkboxes = ref({
  grammar: true,
  html_formatting: true
})
const fetchStatus = ref('init')
const result = ref<IResult | null>(null);

const downloadFile = async (evt: UploadFileInfo): Promise<IResult | void> => {
  fetchStatus.value = 'loading'
  try {
    const formData = new FormData();
    // @ts-ignore
    formData.append('file', evt?.file?.file);
    formData.append('grammar', checkboxes.value.grammar.toString());
    formData.append('html_formatting', checkboxes.value.html_formatting.toString());

    const res = await axios.post('https://13-50-75-136.nip.io/api/v1/ml/file', formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      }
    });

    if (res.data) {
      result.value = res.data
      setTimeout(() => {
        fetchStatus.value = 'success'
      }, 1000)
    }
  } catch (e) {
    fetchStatus.value = 'error'
    console.log(e)
  }
};
const sendText = async (): Promise<IResult | void> => {
  fetchStatus.value = 'loading'
  try {
    const res = await axios.post('https://13-50-75-136.nip.io/api/v1/ml/text', {
      text: text.value,
      grammar: checkboxes.value.grammar,
      html_formatting: checkboxes.value.html_formatting
    });

    if (res.data) {
      result.value = res.data
      setTimeout(() => {
        fetchStatus.value = 'success'
      }, 1000)
    }
  } catch (e) {
    fetchStatus.value = 'error'
    console.log(e)
  }
}

const sendNewRequest =  () => {
  fetchStatus.value = 'init'
  result.value = null
}


watch(fetchStatus, (val) => {
  if (val === 'error') {
    notification.error({
      content: 'Произошла ошибка загрузки',
      duration: 3000
    })
    fetchStatus.value = 'init'
  }
})
</script>

<style lang="scss">
.home-page {
  .n-card {
    border-radius: 1rem;
  }
}
</style>
