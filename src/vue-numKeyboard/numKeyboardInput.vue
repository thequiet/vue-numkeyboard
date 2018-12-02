<template>
<div class="numKeyboardInput" :class="inputClasses">
  <div
        ref="keyboardInput"
        :class="['keyboard-input', { 'keyboard-focus': showKeyboard }, textAlign]"
        @click="focus"
    >
    <!-- wrapper value with span, easy to calc elem length -->
    <span
        v-if="placeholder && value === ''"
        class="placeholder">
        {{ placeholder }}
    </span>
    <span v-else class="input-value">{{ value }}</span>
    <numkeyboard-icon
        name="clear"
        @click.native="clear"
        v-show="showClear && showKeyboard && value"
      >
    </numkeyboard-icon>
  </div>
  <keyboard
      ref="keyboard"
      @typing="typing"
      @clear="clear"
      :show="showKeyboard"
      :activeOk="!!value"
      :point="point"
      :ok-text="okText"
      :value="value"
      :input-label="inputLabel"
      :min-input="minInput"
      :max-input="maxInput">
    </keyboard>
</div>
</template>

<script>
import keyboard from './keyboard'
import { 
  closest, 
  panDetect
} from 'libs/utils'
export default {

  name: 'keyboardInput',

  props: {
    /**
     * raise page height if expose elemenet is covered by keyboard
     * {string} - optional
     * default - keyboardInput
     */
    'exposeElem': String,
    /**
     * digital point display control
     * {boolean} - optional
     * default - true
     */
    'point': {
      type: Boolean,
      default: true
    },
    /**
     * The minimum input allowed
     * {boolean} - optional
     * default - 0
     */
    minInput: {
      type: Number,
    },
    /**
     * The minimum input allowed
     * {boolean} - optional
     * default - 99999
     */
    maxInput: {
      type: Number,
    },
    /**
     * Default value for reset
     * {boolean} - optional
     * default - ''
     */
    'resetValue': {
      type: [String, Number],
      default: ''
    },
    /**
     * Show label for this input
     * {boolean} - optional
     * default - true
     */
    'inputLabel': {
      type: String,
      default: ''
    },
    /**
     * v-model value
     */
    'value': {
      type: [String, Number],
      default: ''
    },
    /**
     * ok button text
     */
    'okText': String,
    /**
     * placeholder
     */
    'placeholder': {
      type: String,
      default: ''
    },
    /**
     * start from left or right
     */
    'textAlign': {
      type: String,
      default: ''
    },
    /**
     * Add classes to input
     */
    'inputClasses': {
      type: String,
    }
  },

  data () {
    return {
      showClear: this.textAlign === 'left',
      showKeyboard: false,
      hasEventCallback: false,
      mutableValue: '',
      orgHeight: '' // original body height property
    };
  },

  methods: {
    typing (code) {
      // set value, insure sync
      let mutableValue = this.value;

      if (code === 'D') { // del
        mutableValue = String(mutableValue).substring(0, String(mutableValue).length -1)
        this.$emit('input', mutableValue)
      } else if (code === 'LD') { // long del
        mutableValue = ''
        this.$emit('input', mutableValue)
      } else if (code === 'F' || code === 'K') { // fold or ok
        this.blur()
        this.$emit('onOk', mutableValue)
      } else if (/[0-9\.]/.test(code)) { // normal input
        if (this.minInput && (this.cleanNumber(mutableValue + code) < this.minInput)) {
          this.$emit('error', 'The minimum value is ' + this.minInput);
          return false;
        }
        if (this.maxInput && (this.cleanNumber(mutableValue + code) > this.maxInput)) {
          this.$emit('error', 'The maximum value is ' + this.maxInput);
          return false;
        }
        mutableValue += code
        this.$emit('input', code === '.' ? mutableValue : mutableValue)
      }
    },

    focus(e) {
      e.preventDefault()

      if (!this.showKeyboard) { // avoid dup click input
        this.showKeyboard = true
        // Reset possible zero values so user does not have to key del
        if (Number(this.value) === 0){
          this.$emit('input', '');
        }
        this.adaptPageHeight()
        this.registerEvents()
      }
    },

    blur () {
      if (!this.value || this.value === '.') {
        this.$emit('input', 0);
      } else {
        this.$emit('input', this.cleanNumber(this.value));
      }

      if (this.showKeyboard) {
        this.resetPageHeight()
      }
      this.showKeyboard = false
    },

    cleanNumber(num) {
      if (isNaN(num)) {
        return 0;
      }
      return Number(String(num));
    },

    clear () {
      this.$emit('input', this.resetValue)
    },

    /**
     * raise page height
     */
    adaptPageHeight () {
      const exposeElem = document.querySelector(this.exposeElem) || this.$refs.keyboardInput
      const keyboard = this.$refs.keyboard.$el
      const win = window
      const body = document.body
      const rect = exposeElem.getBoundingClientRect()
      let diffHeight = keyboard.clientHeight - (win.innerHeight - rect.top - exposeElem.clientHeight)
      if (diffHeight > 0) {
        this.orgHeight = body.style.height
        diffHeight += 30 // 30: additional gap
        body.style.height = body.clientHeight + diffHeight + 'px'
        body.style.removeProperty('transition') // scrollTop will fail when height has transition
        win.scrollTo(0, diffHeight);
      }
    },

    /**
     * resetPageHeight
     */
    resetPageHeight () {
      const body = document.body
      body.style.transition = `height 0.2s`
      if (this.orgHeight) {
        body.style.height = this.orgHeight
      } else {
        body.style.removeProperty('height')
      }
    },

    /**
     * register events
     */
    registerEvents () {
      this.registerBlur()
      this.registerPanDown()
    },

    /**
     * blur
     */
    registerBlur () {
      if (!this.hasEventCallback) {
        const callback = (e) => {
          if (!(closest(e.target, '.keyboard') || closest(e.target, '.keyboard-input'))) {
            this.blur()
            this.hasEventCallback = false
            document.removeEventListener('click', callback)
          }
        }
        this.hasEventCallback = true
        document.addEventListener('click', callback, false)
      }
    },

    /**
     * detect touch down
     */
    registerPanDown () {
      panDetect(({ direction, startTarget }, unregister) => {
        if (direction === 'down' && !closest(startTarget, '.keyboard')) {
          this.blur()
          unregister()
        }
      })
    }
  },

  components: {
    keyboard
  }
};
</script>
