<script setup lang="ts">
defineOptions({
  inheritAttrs: false,
})

const props = withDefaults(
  defineProps<{
    /**
     * The model value of the input.
     *
     * @modifiers
     * `v-model="value"`
     *
     * @modifiers
     * `v-model.lazy="value"`
     */
    modelValue?: number

    /**
     * Used internaly to allow .lazy v-model modifiers.
     */
    modelModifiers?: {
      lazy?: boolean
    }

    /**
     * Minimum value allowed when decrementing
     */
    min?: number

    /**
     * Maximum value allowed when decrementing
     */
    max?: number

    /**
     * Sets the stepping interval when clicking up and down spinner buttons
     */
    step?: number

    /**
     * The form input identifier.
     */
    id?: string

    /**
     * The type of input.
     */
    type?: string

    /**
     * The shape of the input.
     */
    shape?: 'straight' | 'rounded' | 'smooth' | 'curved' | 'full'

    /**
     * The label to display for the input.
     */
    label?: string

    /**
     * If the label should be floating.
     */
    labelFloat?: boolean

    /**
     * The placeholder to display for the input.
     */
    placeholder?: string

    /**
     * The icon to display for the input.
     */
    icon?: string

    /**
     * The icon to display for the decrement button.
     */
    iconDecrement?: string

    /**
     * The icon to display for the increment button.
     */
    iconIncrement?: string

    /**
     * The inputmode to use for the input, usually for mobile devices.
     */
    inputmode?: 'numeric' | 'decimal'

    /**
     * Whether the color of the input should change when it is focused.
     */
    colorFocus?: boolean

    /**
     * Whether the input is in a loading state.
     */
    loading?: boolean

    /**
     * Whether the input is in a disabled state.
     */
    disabled?: boolean

    /**
     * An error message or boolean value indicating whether the input is in an error state.
     */
    error?: string | boolean

    /**
     * The size of the input.
     */
    size?: 'sm' | 'md' | 'lg'

    /**
     * The contrast of the input.
     */
    contrast?: 'default' | 'default-contrast' | 'muted' | 'muted-contrast'

    /**
     * Optional CSS classes to apply to the wrapper, label, input, addon, error, and icon elements.
     */
    classes?: {
      /**
       * CSS classes to apply to the wrapper element.
       */
      wrapper?: string | string[]

      /**
       * CSS classes to apply to the outer element.
       */
      outer?: string | string[]

      /**
       * CSS classes to apply to the label element.
       */
      label?: string | string[]

      /**
       * CSS classes to apply to the input element.
       */
      input?: string | string[]

      /**
       * CSS classes to apply to the addon element.
       */
      addon?: string | string[]

      /**
       * CSS classes to apply to the error element.
       */
      error?: string | string[]

      /**
       * CSS classes to apply to the icon element.
       */
      icon?: string | string[]

      /**
       * CSS classes to apply to the buttons wrapper.
       */
      buttons?: string | string[]
    }
  }>(),
  {
    id: undefined,
    modelValue: undefined,
    modelModifiers: () => ({}),
    min: undefined,
    max: undefined,
    step: 1,
    type: 'text',
    size: 'md',
    contrast: 'default',
    inputmode: 'numeric',
    shape: undefined,
    label: undefined,
    icon: undefined,
    iconDecrement: 'lucide:minus',
    iconIncrement: 'lucide:plus',
    error: false,
    placeholder: undefined,
    classes: () => ({}),
  },
)
const emits = defineEmits<{
  'update:modelValue': [value?: number]
}>()
const appConfig = useAppConfig()
const shape = computed(() => props.shape ?? appConfig.nui.defaultShapes?.input)

const shapeStyle = {
  straight: '',
  rounded: 'nui-input-number-rounded',
  smooth: 'nui-input-number-smooth',
  curved: 'nui-input-number-curved',
  full: 'nui-input-number-full',
}
const sizeStyle = {
  sm: 'nui-input-number-sm',
  md: 'nui-input-number-md',
  lg: 'nui-input-number-lg',
}
const contrastStyle = {
  default: 'nui-input-number-default',
  'default-contrast': 'nui-input-number-default-contrast',
  muted: 'nui-input-number-muted',
  'muted-contrast': 'nui-input-number-muted-contrast',
}

function looseToNumber(val: any) {
  const n = Number.parseFloat(val)
  return Number.isNaN(n) ? val : n
}

const value = useVModel(
  props,
  'modelValue',
  (_, val) => {
    emits('update:modelValue', looseToNumber(val))
  },
  {
    passive: true,
  },
)

const inputRef = ref<HTMLInputElement>()
defineExpose({
  /**
   * The underlying HTMLInputElement element.
   */
  el: inputRef,
})

const id = useNinjaId(() => props.id)
const placeholder = computed(() => {
  if (props.loading) {
    return
  }
  if (props.labelFloat) {
    return props.label
  }

  return props.placeholder
})

const floatPrecision = computed(() => {
  if (!Number.isFinite(props.step) || Number.isNaN(props.step)) return 0
  let exp = 1
  let precision = 0
  while (Math.round(props.step * exp) / exp !== props.step) {
    exp *= 10
    precision++
  }
  return precision
})
const floatPrecisionExp = computed(() => 10 ** floatPrecision.value)
const stepAbs = computed(() => Math.abs(props.step))

function clamp(value: number) {
  const rounded =
    Math.round(value * floatPrecisionExp.value) / floatPrecisionExp.value

  return Math.max(
    Math.min(rounded, props.max ?? Number.POSITIVE_INFINITY),
    props.min ?? Number.NEGATIVE_INFINITY,
  )
}

function increment() {
  if (props.disabled) return

  if (value.value === undefined) {
    value.value = 0
    return
  }

  if (typeof value.value === 'number') {
    value.value = clamp(value.value + stepAbs.value)
  }
}

function decrement() {
  if (props.disabled) return

  if (value.value === undefined) {
    value.value = 0
    return
  }

  if (typeof value.value === 'number') {
    value.value = clamp(value.value - stepAbs.value)
  }
}

let incrementInterval: any
function startIncrement() {
  if (props.disabled) return

  increment()
  let i = 0

  incrementInterval = setInterval(() => {
    i++
    increment()
    if (i > 10) {
      clearInterval(incrementInterval)
      incrementInterval = setInterval(increment, 50)
    }
  }, 150)
}
function stopIncrement() {
  clearInterval(incrementInterval)
}

let decrementInterval: any
function startDecrement() {
  if (props.disabled) return

  decrement()
  let i = 0

  decrementInterval = setInterval(() => {
    i++
    decrement()
    if (i > 10) {
      clearInterval(decrementInterval)
      decrementInterval = setInterval(decrement, 50)
    }
  }, 150)
}
function stopDecrement() {
  clearInterval(decrementInterval)
}

onBeforeUnmount(() => {
  clearInterval(incrementInterval)
  clearInterval(decrementInterval)
})

if (process.dev) {
  const slots = useSlots()
  if (props.labelFloat && 'label' in slots) {
    // eslint-disable-next-line no-console
    console.warn(
      '[ninja-ui][base-input-number] The "label-float" property is not compatible with the label slot, use the label property instead.',
    )
  }
}
</script>

<template>
  <div
    class="nui-input-number-wrapper"
    :class="[
      contrastStyle[props.contrast],
      sizeStyle[props.size],
      shape && shapeStyle[shape],
      props.error && !props.loading && 'nui-input-number-error',
      props.loading && 'nui-input-number-loading',
      props.labelFloat && 'nui-input-number-label-float',
      props.icon && 'nui-has-icon',
      props.colorFocus && 'nui-input-number-focus',
      props.classes?.wrapper,
    ]"
  >
    <label
      v-if="
        ('label' in $slots && !props.labelFloat) ||
        (props.label && !props.labelFloat)
      "
      class="nui-input-number-label"
      :for="id"
      :class="props.classes.label"
    >
      <slot name="label">{{ props.label }}</slot>
    </label>
    <div class="nui-input-number-outer" :class="props.classes?.outer">
      <div>
        <input
          v-if="props.modelModifiers.lazy"
          :id="id"
          ref="inputRef"
          v-model.lazy="value"
          :type="props.type"
          v-bind="$attrs"
          class="nui-input-number"
          :class="props.classes.input"
          :placeholder="placeholder"
          :inputmode="props.inputmode"
          :disabled="props.disabled"
        />
        <input
          v-else
          :id="id"
          ref="inputRef"
          v-model="value"
          :type="props.type"
          v-bind="$attrs"
          class="nui-input-number"
          :class="props.classes.input"
          :placeholder="placeholder"
          :inputmode="props.inputmode"
          :disabled="props.disabled"
        />
        <label
          v-if="
            ('label' in $slots && props.labelFloat) ||
            (props.label && props.labelFloat)
          "
          class="nui-label-float"
          :for="id"
          :class="props.classes.label"
        >
          <slot name="label">{{ props.label }}</slot>
        </label>
        <div v-if="props.loading" class="nui-input-number-placeload">
          <BasePlaceload class="nui-placeload" />
        </div>
        <div
          v-if="props.icon"
          class="nui-input-number-icon"
          :class="props.classes.icon"
        >
          <slot name="icon">
            <Icon :name="props.icon" class="nui-input-number-icon-inner" />
          </slot>
        </div>
        <div class="nui-input-number-buttons" :class="props.classes.buttons">
          <button
            type="button"
            aria-label="Decrement"
            @pointerdown="startDecrement"
            @pointerout="stopDecrement"
            @pointerup="stopDecrement"
            :disabled="props.disabled"
          >
            <Icon :name="props.iconDecrement" />
          </button>
          <button
            type="button"
            aria-label="Increment"
            @pointerdown="startIncrement"
            @pointerout="stopIncrement"
            @pointerup="stopIncrement"
            :disabled="props.disabled"
          >
            <Icon :name="props.iconIncrement" />
          </button>
        </div>
      </div>
    </div>
    <span
      v-if="props.error && typeof props.error === 'string'"
      :class="props.classes?.error"
      class="nui-input-number-error-text"
    >
      {{ props.error }}
    </span>
  </div>
</template>
