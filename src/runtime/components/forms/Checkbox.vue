<template>
  <div :class="ui.wrapper">
    <div class="flex items-center h-5">
      <input
        :id="inputId"
        v-model="toggle"
        :name="name"
        :required="required"
        :value="value"
        :disabled="disabled"
        :checked="checked"
        :indeterminate="indeterminate"
        type="checkbox"
        class="form-checkbox"
        :class="inputClass"
        v-bind="attrs"
        @change="onChange"
      >
    </div>
    <div v-if="label || $slots.label" class="ms-3 text-sm">
      <label :for="inputId" :class="ui.label">
        <slot name="label">{{ label }}</slot>
        <span v-if="required" :class="ui.required">*</span>
      </label>
      <p v-if="help" :class="ui.help">
        {{ help }}
      </p>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, toRef, defineComponent } from 'vue'
import type { PropType } from 'vue'
import { twMerge, twJoin } from 'tailwind-merge'
import { useUI } from '../../composables/useUI'
import { useFormGroup } from '../../composables/useFormGroup'
import { mergeConfig } from '../../utils'
import { uid } from '../../utils/uid'
import type { Strategy } from '../../types'
// @ts-expect-error
import appConfig from '#build/app.config'
import { checkbox } from '#ui/ui.config'
import colors from '#ui-colors'

const config = mergeConfig<typeof checkbox>(appConfig.ui.strategy, appConfig.ui.checkbox, checkbox)

export default defineComponent({
  inheritAttrs: false,
  props: {
    id: {
      type: String,
      // A default value is needed here to bind the label
      default: () => uid()
    },
    value: {
      type: [String, Number, Boolean, Object],
      default: null
    },
    modelValue: {
      type: [Boolean, Array],
      default: null
    },
    name: {
      type: String,
      default: null
    },
    disabled: {
      type: Boolean,
      default: false
    },
    checked: {
      type: Boolean,
      default: false
    },
    indeterminate: {
      type: Boolean,
      default: false
    },
    help: {
      type: String,
      default: null
    },
    label: {
      type: String,
      default: null
    },
    required: {
      type: Boolean,
      default: false
    },
    color: {
      type: String as PropType<typeof colors[number]>,
      default: () => config.default.color,
      validator (value: string) {
        return appConfig.ui.colors.includes(value)
      }
    },
    inputClass: {
      type: String,
      default: ''
    },
    class: {
      type: [String, Object, Array] as PropType<any>,
      default: undefined
    },
    ui: {
      type: Object as PropType<Partial<typeof config & { strategy?: Strategy }>>,
      default: undefined
    }
  },
  emits: ['update:modelValue', 'change'],
  setup (props, { emit }) {
    const { ui, attrs } = useUI('checkbox', toRef(props, 'ui'), config, toRef(props, 'class'))

    const { emitFormChange, color, name, inputId } = useFormGroup(props)

    const toggle = computed({
      get () {
        return props.modelValue
      },
      set (value) {
        emit('update:modelValue', value)
      }
    })

    const onChange = (event: Event) => {
      emit('change', event)
      emitFormChange()
    }

    const inputClass = computed(() => {
      return twMerge(twJoin(
        ui.value.base,
        ui.value.rounded,
        ui.value.background,
        ui.value.border,
        ui.value.ring.replaceAll('{color}', color.value),
        ui.value.color.replaceAll('{color}', color.value)
      ), props.inputClass)
    })

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      attrs,
      toggle,
      inputId,
      // eslint-disable-next-line vue/no-dupe-keys
      name,
      // eslint-disable-next-line vue/no-dupe-keys
      inputClass,
      onChange
    }
  }
})
</script>
