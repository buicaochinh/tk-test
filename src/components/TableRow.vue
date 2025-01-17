<template>
  <tr>
    <td>{{ id }}</td>
    <td>{{ errorDescription || 'No Description' }}</td>
    <td>
      <img
        class="product-image"
        :src="productImage !== '' ? productImage : errorImage"
        :alt="productName"
      />
    </td>
    <td>
      <input
        v-model="productNameState"
        :class="{ input: true, 'is-danger': productNameError !== '' }"
        type="text"
        placeholder="Enter Product Name"
      />
      <label :class="{ label: true, error: productNameError !== '' }">
        {{ productNameError }}
      </label>
    </td>
    <td>
      <input
        v-model="skuState"
        :class="{ input: true, 'is-danger': skuError !== '' }"
        type="text"
        placeholder="Enter SKU"
      />
      <label :class="{ label: true, error: skuError !== '' }">
        {{ skuError }}
      </label>
    </td>
    <td>
      <div class="select">
        <select v-model="colorState">
          <option class="select__default" :value="-1" disabled>
            Select Color
          </option>
          <option
            v-for="colorItem in colorList"
            :key="colorItem.id"
            :value="colorItem.id"
          >
            {{ colorItem.name }}
          </option>
        </select>
      </div>
    </td>
  </tr>
</template>

<script lang="ts">
import { defineComponent, watch } from '@vue/runtime-core'
import { ref, PropType } from 'vue'
import { ErrorType, Color } from '../cores/types'
import { ERROR_IMAGE } from '../cores/errors'

import { validateProductName, validateSKU } from '../utils/validator'

export default defineComponent({
  name: 'TableRow',
  props: {
    colorList: {
      type: Array as PropType<Color[]>,
      default: function (): Color[] {
        return []
      },
    },
    id: {
      type: Number,
      default: 0,
    },
    errorDescription: {
      type: String,
      default: '',
    },
    productImage: {
      type: String,
      default: '',
    },
    productName: {
      type: String,
      default: '',
    },
    sku: {
      type: String,
      default: '',
    },
    color: {
      type: Number,
      default: -1,
    },
  },
  emits: ['changeProductName', 'changeSKU', 'changeColor'],
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  setup(props: any, context) {
    // states
    const cloneProps = { ...props }
    const idState = ref<number>(cloneProps.id)
    const productNameState = ref<string>(cloneProps.productName)
    const skuState = ref<string>(cloneProps.sku)
    const colorState = ref<number>(cloneProps.color)
    const productNameError = ref<string>('')
    const skuError = ref<string>('')

    const errorImage = ERROR_IMAGE

    // watch
    watch(productNameState, (newVal: string) => {
      let validateResult: ErrorType = validateProductName(newVal)
      if (validateResult.status === false) {
        productNameError.value = validateResult.message as string
        context.emit('changeProductName', {
          id: idState.value,
          error: productNameError.value,
        })
      } else {
        productNameError.value = ''
        context.emit('changeProductName', {
          id: idState.value,
          name: productNameState.value,
        })
      }
    })
    watch(skuState, (newVal: string) => {
      let validateResult: ErrorType = validateSKU(newVal)
      if (validateResult.status === false) {
        skuError.value = validateResult.message as string
        context.emit('changeSKU', {
          id: idState.value,
          error: productNameError.value,
        })
      } else {
        skuError.value = ''
        context.emit('changeSKU', { id: idState.value, sku: skuState.value })
      }
    })
    watch(colorState, (newVal: number) => {
      context.emit('changeColor', { id: idState.value, color: newVal })
    })
    // methods

    // return
    return {
      productNameState,
      skuState,
      colorState,
      productNameError,
      skuError,
      errorImage,
    }
  },
})
</script>

<style scoped>
.product-image {
  width: 5rem;
}

option.select__default {
  color: #999;
}
.error {
  color: red;
}
</style>
