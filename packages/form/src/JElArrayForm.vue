<template>
  <el-form ref="form" :model="form_" v-bind="formBind__">
    <el-table :data="form_.data"
              v-bind="tableBind__"
              @select="(selection, row) => $emit('select', selection, row)"
              @select-all="(selection) => $emit('select-all',selection)"
              @selection-change="(selection) => $emit('selection-change',selection)"
              @cell-mouse-enter="onCellMouseEnter"
              @cell-mouse-leave="onCellMouseLeave"
              @cell-click="(row, _column, cell, event) => $emit('cell-click', row, _column, cell, event)"
              @cell-dblclick="(row, _column, cell, event) => $emit('cell-dblclick', row, _column, cell, event)"
              @row-click="(row, _column, event) => $emit('row-click', row, _column, event)"
              @row-contextmenu="(row, _column, event) => $emit('row-contextmenu', row, _column, event)"
              @row-dblclick="(row, _column, event) => $emit('row-dblclick', row, _column, event)"
              @header-click="(_column, event) => $emit('header-click', _column, event)"
              @header-contextmenu="(_column, event) => $emit('header-contextmenu', _column, event)"
              @sort-change="({_column, prop, order}) => $emit('sort-change',{_column, prop, order})"
              @filter-change="(filters) => $emit('filter-change', filters)"
              @current-change="(currentRow, oldCurrentRow) => $emit('current-change', currentRow, oldCurrentRow)"
              @header-dragend="(newWidth, oldWidth, _column, event) => $emit('header-dragend', newWidth, oldWidth, _column, event)"
              @expand-change="(row, expanded) => $emit('expand-change', row, expanded)"
              class="j-array-form"
    >
      <template #empty>
        <el-button v-if="appendEnabled"
                   :disabled="appendButtonDisabled__" type="primary" icon="el-icon-plus"
                   circle size="mini"
                   @click="append"
        ></el-button>
        <span v-else>#</span>
      </template>
      <template #append>
        <div v-if="appendEnabled && form_.data.length > 0" style="text-align: center">
          <el-button :disabled="appendButtonDisabled__" type="primary" icon="el-icon-plus"
                     circle size="mini"
                     @click="append"
          ></el-button>
        </div>
      </template>
      <el-table-column header-align="center" align="center" width="50px" fixed="left">
        <template #default="{row, column, $index}">
          <el-button v-if="removeEnabled && row.__buttonVisible__"
                     :disabled="removeButtonDisabled_" type="danger" icon="el-icon-minus"
                     circle size="mini"
                     @click="remove({$index})"
          ></el-button>
          <span v-else class="j-array-form-index" :class="{
                        required: !!column__.required_
                    }">{{$index + 1}}</span>
        </template>
      </el-table-column>
      <j-el-array-form-column :key="column__.key"
                              :column="column__"
                              :parent="$context__"
      >
      </j-el-array-form-column>
    </el-table>
  </el-form>
</template>

<script>
  import {compare} from "../../../src/utils/util";
  import deepMerge from "../../../src/utils/deep-merge";
  import JElArrayFormColumn from "./JElArrayFormColumn";

  export default {
    name: "j-el-array-form",
    components: {JElArrayFormColumn},
    props: {
      column: {
        type: Object,
        required: true
      },
      value: Array,

      height: [String, Number],
      maxHeight: [String, Number],
      stripe: Boolean,
      border: {
        type: Boolean,
        default: true
      },
      size: String,
      fit: {
        type: Boolean,
        default: true
      },

      minRow: Number,
      maxRow: Number,
      appendEnabled: {
        type: Boolean,
        default: true
      },
      removeEnabled: {
        type: Boolean,
        default: true
      },

      $context: Object
    },
    data() {
      return {
        value_: null,
        form_: {
          data: []
        }
      };
    },
    inject: {
      elForm: {
        default: ''
      },
      elFormItem: {
        default: ''
      },
      form: {default: null}
    },
    computed: {
      elFormItemSize__() {
        return (this.elFormItem || {}).elFormItemSize;
      },
      size__() {
        return this.size || this.elFormItemSize__ || (this.$ELEMENT || {}).size;
      },
      $context__() {
        return this.$context || {
          context: this,
          $model: {
            prop: 'value',
            event: 'input'
          },
          'class': null,
          $class: null,
          style: null,
          $style: null,
          attrs: null,
          $attrs: null,
          props: null,
          $props: null,
          domProps: null,
          $domProps: null,
          on: null,
          $on: null,
          nativeOn: null,
          $nativeOn: null,
          directives: null,
          $directives: null,
          scopedSlots: this.$scopedSlots,
          $scopedSlots: this.$scopedSlots,
          slot: null,
          $slot: null,
          key: null,
          $key: null,
          ref: null,
          $ref: null,
          refInFor: null,
          $refInFor: null
        };
      },
      column__() {
        let _column = deepMerge({}, this.column, {
          width: this.column.width > 0 ? `${this.column.width}px` : null,
          minWidth: this.column.minWidth > 0 ? `${this.column.minWidth}px` : null
        });
        // _column.key = uuid();
        if (!_column.renderData || !_column.renderData.children) return _column;

        let _formItemRenderData = _column.renderData.children[0];
        if (_formItemRenderData.tag !== 'el-form-item') {
          throw new Error('The tag must el-form-item');
        }
        let _label = _formItemRenderData.options.props.label;
        let _prop = _formItemRenderData.options.props.prop;
        let _required = _formItemRenderData.options.attrs.required_;
        _column.label = _label;
        _column.prop = _prop;
        _column.required_ = _required;

        if (!_formItemRenderData.options.scopedSlots) {
          _formItemRenderData.options.scopedSlots = {};
        }

        // 去除 formItem label
        _formItemRenderData.options.scopedSlots.label = {
          tag: 'span'
        };
        if (!_formItemRenderData.options.style) {
          _formItemRenderData.options.style = {};
        }
        _formItemRenderData.options.style.marginBottom = '15px';

        _formItemRenderData.options.props.prop = {
          $getValue: ({$context: {data}}) => {
            let {$index} = data.options;
            return `data.${$index}.${_prop}`;
          }
        };
        return _column;
      },
      formBind__() {
        return {
          size: this.size__
        };
      },
      tableBind__() {
        return {
          height: this.height,
          maxHeight: this.maxHeight,
          stripe: this.stripe,
          border: this.border,
          size: this.size__,
          fit: this.fit,
          showHeader: false
        };
      },
      rowCount__() {
        return this.form_.data.length;
      },
      appendButtonDisabled__() {
        return !!this.maxRow && this.rowCount__ >= this.maxRow;
      },
      removeButtonDisabled_() {
        return !!this.minRow && this.rowCount__ <= this.minRow;
      }
    },
    watch: {
      value: {
        immediate: true,
        handler(val) {
          if (!val) return;
          if (compare(val, this.value_)) return;
          this.value_ = val;
          this.form_.data = this.value_.map(v => {
            let _row = {
              __buttonVisible__: false
            };
            _row[this.column__.prop] = v;
            return _row;
          });
        },
        deep: true
      },
      'form_.data': {
        immediate: true,
        handler(val, oldValue) {
          if (!val || !oldValue) return;
          let _value = val.map(v => v[this.column__.prop]);
          this.$emit('input', _value);
        },
        deep: true
      }
    },
    methods: {
      append() {
        if (this.appendButtonDisabled__) return;
        this.form_.data.push([this.column__].reduce((previousValue, currentValue) => {
          if (!currentValue.renderData || !currentValue.renderData.children) return previousValue;
          let _valueType = currentValue.renderData.children[0].options.attrs.valueType_;
          let _defaultValue = currentValue.renderData.children[0].children[0].children[0].options.props.value;
          switch (_valueType) {
            case 'string':
              previousValue[currentValue.prop] = _defaultValue || '';
              break;
            case 'number':
              previousValue[currentValue.prop] = _defaultValue || 0;
              break;
            case 'array':
              previousValue[currentValue.prop] = _defaultValue || [];
              break;
            case 'object':
              previousValue[currentValue.prop] = _defaultValue || {};
              break;
            case 'boolean':
              previousValue[currentValue.prop] = _defaultValue || false;
              break;
            default:
              throw new Error(`不支持的值类型: ${_valueType}`);
          }
          return previousValue;
        }, {__buttonVisible__: false}));
      },
      remove({$index}) {
        if (this.removeButtonDisabled_) return;
        this.form_.data.splice($index, 1);
      },
      onCellMouseEnter(row, column, cell, event) {
        row.__buttonVisible__ = true;
        this.$emit('cell-mouse-enter', row, column, cell, event);
      },
      onCellMouseLeave(row, column, cell, event) {
        row.__buttonVisible__ = false;
        this.$emit('cell-mouse-leave', row, column, cell, event);
      },
      validate(callback) {
        return this.$refs.form.validate(callback);
      },
      validateField(props, callback) {
        return this.$refs.form.validateField(props, callback);
      },
      resetFields() {
        return this.$refs.form.resetFields();
      },
      clearValidate(props) {
        return this.$refs.form.clearValidate(props);
      }
    },
    created() {
      this.form && this.form.addSubForm(this);
    },
    mounted() {
      this.append();
    }
  }
</script>

<style scoped>
  .j-array-form >>> .el-form-item__label::before {
    content: '' !important;
    color: unset !important;
    margin-right: unset !important;
  }

  .j-array-form >>> .j-array-form-index.required::before {
    content: '*';
    color: #F56C6C;
    margin-right: 4px;
  }
</style>
