<template>
  <el-tooltip :content="title" placement="top" :overlay-style="{zIndex: 2001}">
    <div class="img-check-box" @click="toggle">
      <img :src="img">
      <div v-if="sChecked" class="check-item">
        <i class="el-icon-check" :style="{ color: theme.color }" />
      </div>
    </div>
  </el-tooltip>
</template>

<script>
import { mapState } from 'vuex'
const Group = {
  name: 'ImgCheckboxGroup',
  props: {
    multiple: {
      type: Boolean,
      required: false,
      default: false
    },
    defaultValues: {
      type: Array,
      required: false,
      default: () => []
    },
    title: {
      type: String,
      required: false,
      default: ''
    }
  },
  data() {
    return {
      values: [],
      options: []
    }
  },
  provide() {
    return {
      groupContext: this
    }
  },
  watch: {
    'values': function(value) {
      this.$emit('change', value)
      // // 此条件是为解决单选时，触发两次chang事件问题
      // if (!(newVal.length === 1 && oldVal.length === 1 && newVal[0] === oldVal[0])) {
      //   this.$emit('change', this.values)
      // }
    }
  },
  methods: {
    handleChange(option) {
      if (!option.checked) {
        if (this.values.indexOf(option.value) > -1) {
          this.values = this.values.filter(item => item !== option.value)
        }
      } else {
        if (!this.multiple) {
          this.values = [option.value]
          this.options.forEach(item => {
            if (item.value !== option.value) {
              item.sChecked = false
            }
          })
        } else {
          this.values.push(option.value)
        }
      }
    }
  },
  render(h) {
    return h(
      'div',
      {
        attrs: { style: 'display: flex' }
      },
      [this.$slots.default]
    )
  }
}

export default {
  name: 'ImgCheckbox',
  Group,
  props: {
    checked: {
      type: Boolean,
      required: false,
      default: false
    },
    img: {
      type: String,
      required: true
    },
    value: {
      type: String,
      required: true
    },
    title: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      sChecked: this.initChecked()
    }
  },
  computed: {
    ...mapState('setting', ['theme'])
  },
  inject: ['groupContext'],
  watch: {
    'sChecked': function() {
      const option = {
        value: this.value,
        checked: this.sChecked
      }
      this.$emit('change', option)
      const groupContext = this.groupContext
      if (groupContext) {
        groupContext.handleChange(option)
      }
    }
  },
  created() {
    const groupContext = this.groupContext
    if (groupContext) {
      this.sChecked = groupContext.defaultValues.length > 0 ? groupContext.defaultValues.indexOf(this.value) >= 0 : this.sChecked
      groupContext.options.push(this)
    }
  },
  methods: {
    toggle() {
      if (this.groupContext.multiple || !this.sChecked) {
        this.sChecked = !this.sChecked
      }
    },
    initChecked() {
      const groupContext = this.groupContext
      if (!groupContext) {
        return this.checked
      } else if (groupContext.multiple) {
        return groupContext.defaultValues.indexOf(this.value) > -1
      } else {
        return groupContext.defaultValues[0] === this.value
      }
    }
  }
}
</script>

<style lang="scss">
  .img-check-box{
    margin-right: 16px;
    position: relative;
    border-radius: 4px;
    cursor: pointer;
    .check-item{
      position: absolute;
      top: 0;
      right: 0;
      width: 100%;
      padding-top: 15px;
      padding-left: 24px;
      height: 100%;
      // color: $--color-primary;
      font-size: 14px;
      font-weight: bold;
    }
  }
</style>
