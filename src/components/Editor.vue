<script setup lang="ts">
import { onMounted, onUnmounted, ref, toRefs, watch } from 'vue'
import { useDebounceFn, useResizeObserver, useStorage } from '@vueuse/core'
import * as monaco from 'monaco-editor'
import EditorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'
import JSONWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker'
import CSSWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker'
import HTMLWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker'
import TSWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker'
import { StorageName, initialEditorValue, useDarkGlobal } from '../utils'

const props = defineProps<{
  type: string
}>()

const emit
  = defineEmits<(e: 'change', payload: typeof editorValue.value) => void>()

// @ts-expect-error: Monaco stuff
self.MonacoEnvironment = {
  getWorker: (_: string, label: string) => {
    switch (label) {
      case 'json':
        return new JSONWorker()
      case 'css':
      case 'scss':
      case 'less':
        return new CSSWorker()
      case 'html':
      case 'handlebars':
      case 'razor':
        return new HTMLWorker()
      case 'typescript':
      case 'javascript':
        return new TSWorker()
      default:
        return new EditorWorker()
    }
  }
}

const container = ref<HTMLDivElement | null>(null)
const isDark = useDarkGlobal()
const { type } = toRefs(props)

const editorState = useStorage<Record<string, any>>(
  StorageName.EDITOR_STATE,
  {},
)
const editorValue = useStorage<Record<string, any>>(
  StorageName.EDITOR_VALUE,
  initialEditorValue,
)

let editor: monaco.editor.IStandaloneCodeEditor

onMounted(() => {
  editor = monaco.editor.create(container.value!, {
    language: type.value,
    theme: isDark.value ? 'vs-dark' : 'vs',
    minimap: { enabled: false },
  })

  emit('change', editorValue.value)

  editor.onDidChangeModelContent(
    useDebounceFn(() => {
      if (editorValue.value[type.value] !== editor.getValue()!) {
        editorValue.value[type.value] = editor.getValue()!
        emit('change', editorValue.value)
      }
    }, 500),
  )

  // Set values from storage on load
  if (editorValue.value[type.value]) {
    editor.setValue(editorValue.value[type.value])
    editor.restoreViewState(editorState.value[type.value])
  }
})

watch(type, (currentTab, prevTab) => {
  monaco.editor.setModelLanguage(editor.getModel()!, currentTab)
  editorState.value[prevTab] = editor.saveViewState()
  editor.setValue(editorValue.value[currentTab] ?? '')
  if (editorState.value[currentTab]) {
    editor.restoreViewState(editorState.value[currentTab]!)
    editor.focus()
  }
})

watch(isDark, (value) =>
  editor.updateOptions({
    theme: value ? 'vs-dark' : 'vs',
  })
)

const editorObserver = useResizeObserver(container, () => editor.layout())

onUnmounted(() => {
  editor?.dispose()
  editorObserver.stop()
})
</script>

<template>
  <div ref="container" :class="['fa', 'edit-' + type]" style="height: 33.3%; width: calc(100% - 1px);"></div>
</template>
