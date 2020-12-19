<template>
  <el-row>
    <el-col :span="1"><span>lng</span></el-col>
    <el-col :span="8">
      <el-input-number v-model="value_[0]"
                       :precision="6" :step="1" :controls="false" :disabled="disabled" :size="size"
                       style="width: 100%"/>
    </el-col>
    <el-col :span="1" :offset="1"><span>lat</span></el-col>
    <el-col :span="8">
      <el-input-number v-model="value_[1]"
                       :precision="6" :step="1" :controls="false" :disabled="disabled" :size="size"
                       style="width: 100%"/>
    </el-col>
    <el-col :span="4" :offset="1">
      <el-button type="primary" icon="el-icon-map-location" :disabled="disabled" :size="size"
                 @click="dialogVisible_ = true"/>
    </el-col>
    <j-el-dialog :visible.sync="dialogVisible_" title="选择坐标" append-to-body :close-on-click-modal="false">
      <template #default="{height}">
        <el-input v-model="searchKeyword_" placeholder="请输入地址查找相关位置" style="margin-bottom: 15px"/>
        <template v-if="$options.components && $options.components['baidu-map']">
          <baidu-map scroll-wheel-zoom :center="mapCenter" :zoom="mapZoom"
                     @click="onClickMap">
            <bm-view :style="{
                                width: '100%',
                                height: `${height - 55}px`,
                                flex: 1
                             }"
            />
            <bm-local-search :keyword="searchKeyword_" auto-viewport style="display: none"/>
          </baidu-map>
        </template>
        <template v-else>
          <span>坐标拾取器缺少依赖库: vue-baidu-map</span>
        </template>
      </template>
    </j-el-dialog>
  </el-row>
</template>

<script>
  export default {
    name: "j-el-coordinate-picker",

    props: {
      value: Array,
      disabled: Boolean,
      size: String,
      mapCenter: {
        type: Object,
        default() {
          return {
            lng: 116.404050,
            lat: 39.915486
          };
        }
      },
      mapZoom: {
        type: Number,
        default: 15
      }
    },

    data() {
      return {
        value_: [],
        dialogVisible_: false,
        searchKeyword_: ''
      };
    },

    watch: {
      value: {
        immediate: true,
        handler(val) {
          if (!val) return;
          this.value_ = val;
        }
      },
      value_: {
        handler(val) {
          this.$emit('input', val);
        }
      }
    },

    methods: {
      onClickMap(e) {
        this.value_ = [e.point.lng, e.point.lat];
        this.dialogVisible_ = false;
      }
    }
  }
</script>

<style scoped>

</style>
