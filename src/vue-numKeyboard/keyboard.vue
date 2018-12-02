<template>
  <div :class="['keyboard', { hidden: !show}]">
    <div class="background-smoke"></div>
    <div class="input-anchored" :class="{ 'has-label' : inputLabel }">
      <label v-if="inputLabel">{{ inputLabel }}</label>
      <p>{{ value }}</p>
      <numkeyboard-icon
          name="clear"
          v-show="value"
          @click.native="clear"
      >
      </numkeyboard-icon>
    </div>
    <table v-touch:tap="typing" v-touch:long="longTab">
      <tbody>
        <tr>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(1) }">1</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(2) }">2</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(3) }">3</td>
          <td data-code='D' rowspan="2"><numkeyboard-icon name='del' /></td>
        </tr>
        <tr>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(4) }">4</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(5) }">5</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(6) }">6</td>
        </tr>
        <tr>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(7) }">7</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(8) }">8</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(9) }">9</td>
          <td data-code='K' rowspan="2" :class="{active: activeOk}">{{ okText || 'OK' }}</td>
        </tr>
        <tr>
          <td>{{ point && !hasDecimal ? '.' : '' }}</td>
          <td v-bind:class="{ 'is-disabled': unacceptableInput(0) }">0</td>
          <td data-code='F'><numkeyboard-icon name="keyfold" /></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { closest } from 'libs/utils'

export default {

  name: 'keyboard',

  props: ['show', 'point', 'activeOk', 'okText', 'value', 'inputLabel', 'minInput', 'maxInput'],

  methods: {
    typing (e) {
      e.preventDefault()

      const td = closest(e.target, 'td')

      if (!td) return

      const code = td.dataset.code || td.textContent

      if (!code) return

      this.$emit('typing', code)
    },

    longTab (e) {
      e.preventDefault()

      const td = closest(e.target, 'td')

      if (!td) return

      let code = td.dataset.code || td.textContent

      if (!code) return

      if (code === 'D') {
        code = 'LD'
      }

      this.$emit('typing', code)
    },

    clear() {
      this.$emit('clear');
    },

    cleanNumber(num) {
      if (isNaN(num)) {
        return 0;
      }
      return Number(String(num));
    },

    unacceptableInput(num) {
      let result = this.cleanNumber(String(this.value + '' + num));
      if (this.minInput && result < this.minInput) {
        return true;
      }
      if (this.maxInput && result > this.maxInput) {
        return true;
      }
      return false;
    },
  },
  computed: {
    hasDecimal() {
      return String(this.value).includes('.');
    }
  },
};
</script>
