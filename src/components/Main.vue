<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'
import { useStorage } from '@vueuse/core'
import Split from 'split.js'

import { StorageName, generateHTML, useDarkGlobal } from '../utils'
import Editor from './Editor.vue'
import Tabs from './Tabs.vue'

const iframe = ref<HTMLIFrameElement>()

const items = ref([
  { text: 'HTML', value: 'html' },
  { text: 'CSS', value: 'css' },
  { text: 'JS', value: 'javascript' },
])

const isDark = useDarkGlobal()

watch(isDark, (value) => {
  iframe.value?.contentWindow?.postMessage(
    `theme-${value ? 'dark' : 'light'}`,
    '*',
  )
})

const onChange = (payload: Record<string, any>) => {
  iframe.value!.srcdoc = generateHTML(payload, isDark.value)
}

onMounted(() => {
  Split(['#split', 'iframe'])
  Split(['.edit-html', '.edit-css', '.edit-javascript'], {
    direction: 'vertical'
  })
})
</script>

<template>
  <main class="flex-1 h-screen border-t border-gray-200 dark:border-gray-700">
    <div class="flex flex-row h-full">
      <div id="split" class="w-full flex flex-col mt-3">
        <Editor type="html" @change="onChange" />
        <Editor type="css" @change="onChange" />
        <Editor type="javascript" @change="onChange" />
      </div>
      <iframe
        ref="iframe"
        class="h-full w-full border-[#393939] border-l border"
        sandbox="allow-scripts"
        frameBorder="0"
      ></iframe>
    </div>
  </main>
</template>

<style>
main {
  height: calc(100vh - var(--nav-height));
}

.gutter {
  @apply bg-no-repeat border-transparent;
  background-position: 50%;
}

.gutter.gutter-horizontal {
  cursor: col-resize;
}

.gutter.gutter-vertical {
  cursor: row-resize;
}
</style>
