<template>
  <div class="text-editor">
    <quill-editor v-model:content="content" theme="snow" :modules="modules" content-type="html" toolbar="essential" style="min-height: 600px;"></quill-editor>
  </div>
</template>

<script setup lang="ts">
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css';
import htmlEditButton from "quill-html-edit-button";
import {ref, watch} from "vue";

const modules = [
  {
    name: 'htmlEditButton',
    module: htmlEditButton,
    options: {
      msg: 'Редактиование HTML разметки',
      okText: 'Сохранить',
      cancelText: 'Отмена',
    }
  }
]

const props = defineProps<{html: string}>()
const emit = defineEmits(['update:html'])


const content = ref(props.html)
watch(content, () => {
  emit('update:html', content.value)
}, { immediate: true })

</script>

<style lang="scss">
.ql-toolbar {
  border-radius: 1rem 1rem 0 0;
}
.ql-container {
  border-radius: 0 0 1rem 1rem;
}
.ql-html-popupTitle {
  font-size: 2rem;
  font-style: initial!important;
  margin-bottom: 1rem!important;
}
.ql-html-textArea {
  border-radius: 1rem;
  padding-bottom: 2rem;
}
.ql-html-buttonGroup {
  button {
    cursor: pointer;
  }
}

.ql-editor {
  strong {
    font-weight: bold!important;
  }
  em {
    font-style: italic!important;
  }
  u {
    text-decoration: underline;
  }
}
</style>
