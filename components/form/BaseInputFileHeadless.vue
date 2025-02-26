<script setup lang="ts">
import { useNinjaFilePreview } from '../../composables/file-preview'

defineOptions({
  inheritAttrs: false,
})

const props = withDefaults(
  defineProps<{
    /**
     * The model value of the file input.
     */
    modelValue?: FileList | null

    /**
     * The form input identifier.
     */
    id?: string

    /**
     * Allows multiple files to be selected.
     */
     multiple?: boolean

    /**
     * Allows to filter files when dropped.
     */
    filterFileDropped?: (file: File) => boolean
  }>(),
  {
    id: undefined,
    modelValue: undefined,
    multiple: false,
    filterFileDropped: () => true,
  },
)
const emits = defineEmits<{
  'update:modelValue': [value?: FileList | null]
}>()
const inputRef = ref<HTMLInputElement>()
const value = useVModel(props, 'modelValue', emits, {
  passive: true,
})

const id = useNinjaId(() => props.id)

const previewMap = new WeakMap<File, Ref<string | undefined>>()

function open() {
  inputRef.value?.click()
}
function drop(event: DragEvent) {
  event.stopPropagation()
  event.preventDefault()

  const dt = event.dataTransfer
  const filtered = new DataTransfer()
  if (inputRef.value && dt) {
    for (const file of dt.files) {
      if (props.filterFileDropped(file)) {
        filtered.items.add(file)
      }
    }
    inputRef.value.files = filtered.files
    value.value = inputRef.value.files
  }
}
function remove(file?: File) {
  if (!file) return
  if (!value.value) return
  if (!inputRef.value) return

  const filtered = new DataTransfer()

  if (previewMap.has(file)) {
    previewMap.delete(file)
  }

  for (const f of value.value) {
    if (f !== file) {
      filtered.items.add(f)
    }
  }

  inputRef.value.files = filtered.files
  value.value = inputRef.value.files
}

function handleFileChange(event: Event) {
  const newFiles = (event.target as HTMLInputElement).files;
  if (!newFiles) return;

  if (props.multiple && value.value) {
    // When multiple is true, append new files to existing ones
    const existingFiles = Array.from(value.value);
    const updatedFiles = new DataTransfer();

    // Add all existing files
    for (const file of existingFiles) {
      updatedFiles.items.add(file);
    }

    // Add new files, optionally check for duplicates
    for (const newFile of newFiles) {
      if (!existingFiles.some(existingFile => existingFile.name === newFile.name)) {
        updatedFiles.items.add(newFile);
      }
    }
    if (!inputRef.value) return

    inputRef.value.files = updatedFiles.files;
    value.value = updatedFiles.files;
  } else {
    // When multiple is false, replace current files with new selection
    value.value = newFiles;
  }
}

provide(
  'BaseInputFileHeadlessContext',
  reactive({
    el: inputRef,
    id,
    files: value,
    open,
    remove,
    preview: useNinjaFilePreview,
    drop,
  }),
)

defineExpose({
  /**
   * The underlying HTMLInputElement element.
   */
  el: inputRef,
  /**
   * The form input identifier.
   */
  id,
  /**
   * The model value of the file input.
   */
  files: value,
  /**
   * Opens the native file input selector.
   */
  open,
  /**
   * Removes a file from the input.
   */
  remove,
  /**
   * Returns the preview DataURL of a file.
   */
  preview: useNinjaFilePreview,
  /**
   * Handles the drop event.
   */
  drop,
})
</script>

<template>
  <div class="group/nui-file-headless relative">
    <slot
      :id="id"
      :el="inputRef"
      :files="value"
      :open="open"
      :remove="remove"
      :preview="useNinjaFilePreview"
      :drop="drop"
    ></slot>
    <input
      :id="id"
      ref="inputRef"
      type="file"
      v-bind="$attrs"
      class="hidden"
      :multiple="props.multiple"
      @change="handleFileChange"
    />
  </div>
</template>
