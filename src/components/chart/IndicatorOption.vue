<template>
  <div class="indicator-option form-group" :class="['-' + type]">
    <label>
      {{ label }}
    </label>

    <dropdown
      v-if="name === 'lineType'"
      class="-left -center"
      :selected="value"
      :options="{ 0: 'Simple', 1: 'with steps' }"
      selectionClass="-outline form-control -arrow"
      placeholder="lineType"
      @output="setValue($event)"
    ></dropdown>
    <dropdown
      v-else-if="/linestyle$/i.test(name)"
      class="-left -center"
      :selected="value"
      :options="{ 0: 'Solid', 1: 'Dotted', 2: 'Dashed', 3: 'LargeDashed', 4: 'SparseDotted' }"
      selectionClass="-outline form-control -arrow"
      placeholder="lineStyle"
      @output="setValue($event)"
    ></dropdown>
    <verte
      v-else-if="type === 'color'"
      picker="square"
      menuPosition="left"
      :label="label"
      model="rgb"
      :value="value"
      @input="setValue($event)"
      @close="reloadIndicator"
    ></verte>
    <editable v-else-if="type === 'string' || type === 'number'" class="form-control" :content="value" @output="setValue($event)"></editable>
    <label v-else-if="type === 'boolean'" class="checkbox-control">
      <input type="checkbox" class="form-control" :checked="value" @change="setValue($event.target.checked)" />
      <span>{{ label }}</span>
      <div></div>
    </label>
  </div>
</template>
<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'
import { getDefaultIndicatorOptionValue } from './chartOptions'

@Component({
  name: 'IndicatorOption',
  props: {
    indicatorId: {
      type: String,
      required: true
    },
    paneId: {
      type: String,
      required: true
    },
    plotTypes: {
      required: true
    },
    name: {
      type: String,
      required: true
    }
  }
})
export default class extends Vue {
  private indicatorId: string
  private paneId: string
  private plotTypes: string[]
  private name: string

  value = null

  get currentIndicatorValue() {
    return this.$store.state[this.paneId].indicators[this.indicatorId].options[this.name]
  }

  get label() {
    return this.name
  }

  get type() {
    let type = 'string'

    let typedValue

    try {
      typedValue = JSON.parse(this.value)
    } catch (error) {
      typedValue = this.value
      // empty
    }

    if (typeof typedValue === 'boolean' || /^(show|toggle|set|use)[A-Z]/.test(this.name)) {
      type = 'boolean'
    } else if (/color/i.test(this.name) || /^rgba?/.test(typedValue) || /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/.test(typedValue)) {
      type = 'color'
    } else if (typeof typedValue === 'number') {
      type = 'number'
    }

    return type
  }

  @Watch('currentIndicatorValue')
  onIndicatorValueChange() {
    const value = this.getValue()

    if (+this.value !== +value || (isNaN(+this.value) && value !== this.value)) {
      this.value = value
    }
  }

  created() {
    this.value = this.getValue()
  }

  getValue() {
    let preferedValue

    if (typeof this.currentIndicatorValue !== 'undefined') {
      preferedValue = this.currentIndicatorValue
    }

    const defaultValue = getDefaultIndicatorOptionValue(this.name, this.plotTypes)

    if (typeof preferedValue !== 'undefined') {
      if (preferedValue && typeof preferedValue === 'object' && defaultValue && typeof defaultValue === 'object') {
        return Object.assign({}, defaultValue, preferedValue)
      } else {
        return preferedValue
      }
    } else if (typeof defaultValue !== 'undefined') {
      return defaultValue
    }

    return null
  }

  setValue(value) {
    this.$store.dispatch(this.paneId + '/setIndicatorOption', {
      id: this.indicatorId,
      key: this.name,
      value
    })

    this.value = value
  }

  reloadIndicator() {
    this.$store.commit(this.paneId + '/SET_INDICATOR_SCRIPT', {
      id: this.indicatorId,
      value: this.$store.state[this.paneId].indicators[this.indicatorId].script
    })
  }
}
</script>
<style lang="scss">
.indicator-option {
  > label {
    display: block;
  }

  &.-boolean,
  &.-color {
    display: flex;
    align-items: center;

    > label:first-child {
      margin: 0 1rem 0 0;
      flex-grow: 1;
    }
  }

  &.-color > label {
    margin: 0 1rem 0 0;
    flex-grow: 1;
  }

  &.-boolean {
    > label:first-child {
      display: none;
    }

    .checkbox-control {
      flex-grow: 1;

      > span {
        flex-grow: 1;
        word-break: break-word;
      }
    }
  }

  + .indicator-option {
    margin-top: 1rem;
  }

  .form-control {
    width: 100%;
    max-width: 200px;
  }
}
</style>
