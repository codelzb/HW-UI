<template>
  <div class="h-input" :class="{error}">
    <input :value="value" type="text" :disabled="disabled" :readonly="readonly"
      @change="$emit('change', $event.target.value)"
      @input="$emit('input', $event.target.value)"
      @focus="$emit('focus', $event.target.value)"
      @blur="$emit('blur', $event.target.value)"
    >
    <template v-if="error">
      <icon name="error" class="icon-error"></icon>
      <span class="errorMessage">{{error}}</span>
    </template>
  </div>
</template>
<script>
  import Icon from './icon'
  export default {
    components: {Icon},
    name: 'hInput',
    props: {
      value: {
        type: String
      },
      disabled: {
        type: Boolean,
        default: false
      },
      readonly: {
        type: Boolean,
        default: false
      },
      error: {
        type: String
      }
    }
  }
</script>
<style lang="scss">
  $height: 32px;
  $border-color: #c2cedf;
  $border-color-hover: #66b1ff;
  $border-radius: 4px;
  $font-size: 12px;
  $red: #F1453D;
  .h-input { font-size: $font-size; display: inline-flex;
    align-items: center;line-height:$font-size;
    > :not(:last-child) {margin-right: .5em; }
    > input { height: 32px; border: 1px solid $border-color; border-radius: 4px; padding: 0 8px; font-size: inherit;
      &:hover { border-color: $border-color-hover; }
      &:focus {  outline: none; }
      &[disabled], &[readonly] {border-color: #bbb;color: #bbb;cursor: not-allowed; }
    }
    &.error {
      > input { border-color: $red; }
    }
    .icon-error { fill: $red; }
    .errorMessage { color: $red; }
  }
</style>