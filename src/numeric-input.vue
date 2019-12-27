<template>
  <input
    v-model="displayText"
    type="text"
    ref="input"
    @input="onInput"
    @focus="onFocus"
    @keydown="onKeydown"
    @blur="onBlur"
    style="text-align: right"/>
</template>

<script>
import Big from 'big.js'
import toFormat from 'toformat'

toFormat(Big)

export default {
  props: {
    value: {
      type: String
    },

    editing: {
      type: Boolean,
      default: true
    },

    decimalPlaces: {
      type: Number,
      default: 2
    },

    minValue: {
      type: String,
      required: false
    },

    maxValue: {
      type: String,
      required: false
    },

    displayNullAs: {
      type: String,
      default: ''
    },

    displayZeroAsBlank: {
      type: Boolean,
      default: false
    },

    displayDecimalsIfZero: {
      type: Boolean,
      default: true
    }
  },

  data() {
    return {
      displayText: ''
    }
  },

  watch: {
    value() {
      this.updateDisplayText()
    }
  },

  created() {
    this.updateDisplayText()
  },

  computed: {
    currentValue() {
      return this.getValue(this.displayText)
    },

    formatedValue() {
      return this.formatValue(this.currentValue)
    }
  },

  methods: {
    updateDisplayText() {
      if (document.activeElement != this.$refs.input || this.getValue(this.value) != this.currentValue)
        this.displayText = this.formatValue(this.value)
    },

    getValue(text) {
      if (text != '' && text != null && text != '-' && text != '.') {
        let strippedText = text.replace(/[^(0-9\.\-)]/g, '').replace(/\.0*$/,'').toString()
        return Big(strippedText).round(this.decimalPlaces).toString()
      } else
        return null
    },

    formatValue(value) {
      if (value == '' || value == null)
        return ''
      else if (value == '0' && this.displayZeroAsBlank)
        return ''
      else
        return new Big(value).toFormat(this.decimalPlaces)
    },

    validNumberInput(text) {
      if (text == '-' || text == '.')
        return true
      else if  (/^[-]?([0-9]+([.][0-9]*)?|[.][0-9]+)$/.test(text))
        return true
      else
        return false
    },

    predictInput(text, cursor, char) {
      let chars = text.split('')
      chars.splice(cursor, 0, char)
      return chars.join('')
    },

    onInput() {
      this.$emit('input', this.currentValue)
    },

    onFocus(event) {
      if (this.editing) {
        event.target.value = this.getValue(event.target.value)
      }
      event.target.select()
    },

    onKeydown(event) {
      // check key.length to verify printable char
      if (event.key && event.key.length == 1) {
        if ('0123456789.-'.includes(event.key)) {
          if (!this.validNumberInput(this.predictInput(event.target.value, event.target.selectionStart, event.key))) {
            event.preventDefault()
          }
        } else {
          event.preventDefault()
        }
      }
    },

    onBlur(event) {
      this.displayText = this.formatedValue
    }
  }
}
</script>
