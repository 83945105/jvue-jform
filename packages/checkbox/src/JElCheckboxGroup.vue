<template>
  <el-checkbox-group v-model="value_" v-bind="bind__" @change="val => $emit('change', val)">
    <template v-for="(option, $index) in data_">
      <el-checkbox-button v-if="mode === 'button'"
                          :key="$index"
                          :label="option.value"
                          :disabled="option.disabled">{{option.label}}
      </el-checkbox-button>
      <el-checkbox v-else
                   :key="$index"
                   :label="option.value"
                   :disabled="option.disabled"
                   :border="option.border|| border">{{option.label}}
      </el-checkbox>
    </template>
  </el-checkbox-group>
</template>

<script>
  import {isObject} from "../../../src/utils/util";

  export default {
    name: "j-el-checkbox-group",

    props: {
      value: Array,                       // 绑定值
      size: String,                       // 单选框组尺寸，仅对按钮形式的 Radio 或带有边框的 Radio 有效
      disabled: Boolean,                  // 是否禁用
      min: Number,                        // 可被勾选的 checkbox 的最小数量
      max: Number,                        // 可被勾选的 checkbox 的最大数量
      textColor: {                        // 按钮形式的 Radio 激活时的文本颜色
        type: String,
        default: '#ffffff'
      },
      fill: {                             // 按钮形式的 Radio 激活时的填充色和边框色
        type: String,
        default: '#409EFF'
      },

      border: Boolean,                    // 是否显示边框
      mode: {                             // 模式
        type: String,
        validator(value) {
          return [null, 'button'].indexOf(value) !== -1
        }
      },

      data: Array,                        // 选项数据
      dataProp: {                         // 数据属性
        type: Object,
        default() {
          return {
            labelKey: 'label',                  // 用于解析选项标签的key
            valueKey: 'value',                  // 用于解析选项值的key
            trueLabelKey: 'trueLabel',          // 选中时的值
            falseLabelKey: 'falseLabel',        // 没有选中时的值
            disabledKey: 'disabled',            // 用于解析选项是否禁用的key
            borderKey: 'border',                // 用于解析选项是否有边框的key
            checkedKey: 'checked',              // 当前是否勾选
            indeterminateKey: 'indeterminate'   // 设置 indeterminate 状态，只负责样式控制
          };
        },
        validator(value) {
          return isObject(value);
        }
      }
    },

    data() {
      return {
        value_: [],
        data_: []
      };
    },

    computed: {
      bind__() {
        return {
          size: this.size,
          disabled: this.disabled,
          min: this.min,
          max: this.max,
          textColor: this.textColor,
          fill: this.fill
        };
      }
    },

    watch: {
      value: {
        immediate: true,
        handler(val) {
          this.value_ = val;
        }
      },
      value_: {
        immediate: true,
        handler(val) {
          this.$emit('input', val);
        }
      },
      data: {
        immediate: true,
        handler(val) {
          this.data_ = (val || []).map(option => {
            return {
              label: option[this.dataProp.labelKey],
              value: option[this.dataProp.valueKey],
              trueLabel: option[this.dataProp.trueLabelKey],
              falseLabel: option[this.dataProp.falseLabelKey],
              disabled: !!option[this.dataProp.disabledKey],
              border: !!option[this.dataProp.borderKey],
              checked: !!option[this.dataProp.checkedKey],
              indeterminate: !!option[this.dataProp.indeterminateKey]
            };
          });
        }
      }
    }
  }
</script>

<style scoped>

</style>
