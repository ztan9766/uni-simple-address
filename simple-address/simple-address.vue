<template>
  <view class="simple-address" v-if="showPopup" @touchmove.stop.prevent="clear">
    <!-- 遮罩层 -->
    <view class="simple-address-mask" @touchmove.stop.prevent="clear" v-if="maskClick"
      :class="[ani + '-mask', animation ? 'mask-ani' : '']" :style="{
				'background-color': maskBgColor
			}" @tap="hideMask(true)"></view>

    <view class="simple-address-content simple-address--fixed"
      :class="[type, ani + '-content', animation ? 'content-ani' : '']">
      <view class="simple-address__header">
        <view class="simple-address__header-btn-box" @click="pickerCancel">
          <text class="simple-address__header-text" :style="{ color: cancelColor, fontSize: btnFontSize }">取消</text>
        </view>
        <view class="simple-address__header-btn-box" @click="pickerConfirm">
          <text class="simple-address__header-text"
            :style="{ color: confirmColor || themeColor, fontSize: btnFontSize }">确定</text>
        </view>
      </view>
      <view class="simple-address__box">
        <picker-view indicator-style="height: 70rpx;" class="simple-address-view" :value="pickerValue"
          @change="pickerChange">
          <picker-view-column>
            <!-- #ifndef APP-NVUE -->
            <view class="picker-item" :style="{ fontSize: fontSize }" v-for="(item, index) in provinceDataList"
              :key="index">{{ provinceData[item] }}</view>
            <!-- #endif -->
            <!-- #ifdef APP-NVUE -->
            <text class="picker-item" :style="{ fontSize: fontSize }" v-for="(item, index) in provinceDataList"
              :key="index">{{ provinceData[item] }}</text>
            <!-- #endif -->
          </picker-view-column>
          <picker-view-column>
            <!-- #ifndef APP-NVUE -->
            <view class="picker-item" :style="{ fontSize: fontSize }" v-for="(item, index) in cityDataList"
              :key="index">{{ cityData[item] }}</view>
            <!-- #endif -->
            <!-- #ifdef APP-NVUE -->
            <text class="picker-item" :style="{ fontSize: fontSize }" v-for="(item, index) in cityDataList"
              :key="index">{{ cityData[item] }}</text>
            <!-- #endif -->
          </picker-view-column>
          <picker-view-column>
            <!-- #ifndef APP-NVUE -->
            <view class="picker-item" :style="{ fontSize: fontSize }" v-for="(item, index) in areaDataList"
              :key="index">{{ areaData[item] }}</view>
            <!-- #endif -->
            <!-- #ifdef APP-NVUE -->
            <text class="picker-item" :style="{ fontSize: fontSize }" v-for="(item, index) in areaDataList"
              :key="index">{{ areaData[item] }}</text>
            <!-- #endif -->
          </picker-view-column>
        </picker-view>
      </view>
    </view>
  </view>
</template>

<script>
/**
 * Simple-addres  地址联动组件
 * @description 三级地址联动，支持（app）nvue、小程序、H5
 * @tutorial https://ext.dcloud.net.cn/plugin?id=1084
 * @property {String} animation 是否开启动画
 * @property {String} type = [bottom] 弹出层类型，暂时只支持底部弹出
 * @property {Boolean} maskClick = [true | false] 是否允许点击遮罩层关闭
 * @property {Boolean} show = [true | false]  显示或隐藏地址组件
 * @property {String} maskBgColor 遮罩层背景颜色
 * @property {String} cancelColor 取消按钮颜色，默认为：#1aad19
 * @property {String} confirmColor 确认按钮颜色，默认为：themeColor
 * @property {String} themeColor 主题颜色，后续会废弃该配置，建议使用`cancelColor`或`confirmColor`
 * @property {String} btnFontSize 取消、确认按钮字体大小，默认为`uni.scss里的 $uni-font-size-base `
 * @property {String} fontSize picker-item字体大小，默认为：28rpx
 * @property {Array} pickerValueDefault 默认值，可以通过function queryIndex 获取
 * @property {Function} queryIndex 根据自定义信息返回对应的index
 * @property {Function} open 打开
 * @example  <simple-address ref="simpleAddress" :pickerValueDefault="cityPickerValueDefault" @onConfirm="onConfirm" themeColor='#007AFF'></simple-address>
 */

import pcaa from './city-data/pcaa'

export default {
  name: 'simpleAddress',
  props: {
    mode: {
      // 地址类型
      // default 则代表老版本根据index索引获取数据
      //
      type: String,
      default: 'default',
    },
    // 开启动画
    animation: {
      type: Boolean,
      default: true,
    },
    /* 弹出层类型，可选值；
				bottom：底部弹出层
			*/
    type: {
      type: String,
      default: 'bottom',
    },
    // maskClick
    maskClick: {
      type: Boolean,
      default: true,
    },
    show: {
      type: Boolean,
      default: true,
    },
    maskBgColor: {
      type: String,
      default: 'rgba(0, 0, 0, 0.4)', //背景颜色 rgba(0, 0, 0, 0.4) 为空则调用 uni.scss
    },
    themeColor: {
      type: String,
      default: '', // 确认按钮颜色（向下兼容）
    },
    cancelColor: {
      type: String,
      default: '', // 取消按钮颜色
    },
    confirmColor: {
      type: String,
      default: '', // 确认按钮颜色
    },
    fontSize: {
      type: String,
      default: '28rpx', // picker-item字体大小
    },
    btnFontSize: {
      type: String,
      default: '', // 按钮的字体大小
    },
    /* 默认值 */
    pickerValueDefault: {
      type: Array,
      default() {
        return [0, 0, 0]
      },
    },
  },
  data() {
    return {
      ani: '',
      showPopup: false,
      pickerValue: [0, 0, 0],
      provinceData: {},
      cityData: {},
      areaData: {},
    }
  },
  watch: {
    show(newValue) {
      if (newValue) {
        this.open()
      } else {
        this.close()
      }
    },
    pickerValueDefault() {
      this.init()
    },
  },
  created() {
    this.init()
  },
  computed: {
    provinceDataList() {
      return Object.keys(this.provinceData)
    },
    cityDataList() {
      return Object.keys(this.cityData)
    },
    areaDataList() {
      return Object.keys(this.areaData)
    },
  },
  methods: {
    init() {
      this.handPickValueDefault() // 对 pickerValueDefault 做兼容处理
      this.provinceData = pcaa['86']
      this.cityData = pcaa[this.provinceDataList[this.pickerValueDefault[0]]]
      this.areaData = pcaa[this.cityDataList[this.pickerValueDefault[1]]]
      this.pickerValue = this.pickerValueDefault
    },
    handPickValueDefault() {
      if (this.pickerValueDefault !== [0, 0, 0]) {
        const provinceKeyList = Object.keys(pcaa['86'])
        if (this.pickerValueDefault[0] > provinceKeyList.length - 1) {
          this.pickerValueDefault[0] = provinceKeyList.length - 1
        }
        const cityKeyList = Object.keys(pcaa[provinceKeyList[this.pickerValueDefault[0]]])
        if (this.pickerValueDefault[1] > cityKeyList.length - 1) {
          this.pickerValueDefault[1] = cityKeyList.length - 1
        }
        const areaKeyList = Object.keys(pcaa[cityKeyList[this.pickerValueDefault[1]]])
        if (this.pickerValueDefault[2] > areaKeyList.length - 1) {
          this.pickerValueDefault[2] = areaKeyList.length - 1
        }
      }
    },
    pickerChange(e) {
      let changePickerValue = e.detail.value
      if (this.pickerValue[0] !== changePickerValue[0]) {
        // 第一级发生滚动
        this.cityData = pcaa[this.provinceDataList[changePickerValue[0]]]
        this.areaData = pcaa[this.cityDataList[changePickerValue[1]]]
        changePickerValue[1] = 0
        changePickerValue[2] = 0
      } else if (this.pickerValue[1] !== changePickerValue[1]) {
        // 第二级滚动
        this.areaData = pcaa[this.cityDataList[changePickerValue[1]]]
        changePickerValue[2] = 0
      }
      this.pickerValue = changePickerValue
      this._$emit('onChange')
    },
    _$emit(emitName) {
      let pickObj = {
        label: this._getLabel(),
        value: this.pickerValue,
        cityCode: this._getCityCode(),
        areaCode: this._getAreaCode(),
        provinceCode: this._getProvinceCode(),
        labelArr: this._getLabel().split('-'),
      }
      this.$emit(emitName, pickObj)
    },
    _getLabel() {
      let pickerLabel =
        this.provinceData[this.provinceDataList[this.pickerValue[0]]] +
        '-' +
        this.cityData[this.cityDataList[this.pickerValue[1]]] +
        '-' +
        this.areaData[this.areaDataList[this.pickerValue[2]]]
      return pickerLabel
    },
    _getProvinceCode() {
      return this.provinceDataList[this.pickerValue[0]]
    },
    _getCityCode() {
      return this.cityDataList[this.pickerValue[1]]
    },
    _getAreaCode() {
      return this.areaDataList[this.pickerValue[2]]
    },
    queryIndex(params = []) {
      if (params === [] || params.length !== 3) {
        return [0, 0, 0]
      }
      const provinceKeyList = Object.keys(pcaa['86'])
      const provinceIndex = provinceKeyList.indexOf(params[0]) > -1 ? provinceKeyList.indexOf(params[0]) : 0

      const cityKeyList = Object.keys(pcaa[provinceKeyList[provinceIndex]])
      const cityIndex = cityKeyList.indexOf(params[1]) > -1 ? cityKeyList.indexOf(params[1]) : 0

      const areaKeyList = Object.keys(pcaa[cityKeyList[cityIndex]])
      const areaIndex = areaKeyList.indexOf(params[2]) > -1 ? areaKeyList.indexOf(params[2]) : 0

      const provinceCode = provinceKeyList[provinceIndex]
      const provinceLabel = pcaa['86'][provinceCode]
      const cityCode = cityKeyList[cityIndex]
      const cityLabel = pcaa[provinceCode][cityCode]
      const areaCode = areaKeyList[areaIndex]
      const areaLabel = pcaa[cityCode][areaCode]

      return {
        index: [provinceIndex, cityIndex, areaIndex],
        data: {
          province: {
            label: provinceLabel,
            value: provinceCode,
          },
          city: {
            label: cityLabel,
            value: cityCode,
          },
          area: {
            label: areaLabel,
            value: areaCode,
          },
        },
      }
    },
    clear() {},
    hideMask() {
      this._$emit('onCancel')
      this.close()
    },
    pickerCancel() {
      this._$emit('onCancel')
      this.close()
    },
    pickerConfirm() {
      this._$emit('onConfirm')
      this.close()
    },
    open() {
      this.showPopup = true
      this.$nextTick(() => {
        setTimeout(() => {
          this.ani = 'simple-' + this.type
        }, 100)
      })
    },
    close(type) {
      if (!this.maskClick && type) return
      this.ani = ''
      this.$nextTick(() => {
        setTimeout(() => {
          this.showPopup = false
        }, 300)
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.simple-address {
  /* #ifndef APP-NVUE */
  display: flex;
  /* #endif */
  flex-direction: column;
}

.simple-address-mask {
  position: fixed;
  bottom: 0;
  top: 0;
  left: 0;
  right: 0;

  transition-property: opacity;
  transition-duration: 0.3s;
  opacity: 0;
  /* #ifndef APP-NVUE */
  z-index: 99;
  /* #endif */
}

.mask-ani {
  transition-property: opacity;
  transition-duration: 0.2s;
}

.simple-bottom-mask {
  opacity: 1;
}

.simple-center-mask {
  opacity: 1;
}

.simple-address--fixed {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  transition-property: transform;
  transition-duration: 0.3s;
  transform: translateY(460rpx);
  /* #ifndef APP-NVUE */
  z-index: 99;
  /* #endif */
}

.simple-address-content {
  background-color: #ffffff;
}

.simple-content-bottom {
  bottom: 0;
  left: 0;
  right: 0;
  transform: translateY(500rpx);
}

.content-ani {
  transition-property: transform, opacity;
  transition-duration: 0.2s;
}

.simple-bottom-content {
  transform: translateY(0);
}

.simple-center-content {
  transform: scale(1);
  opacity: 1;
}

.simple-address__header {
  position: relative;
  /* #ifndef APP-NVUE */
  display: flex;
  /* #endif */
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-between;
  border-bottom-color: #f2f2f2;
  border-bottom-style: solid;
  border-bottom-width: 1rpx;
}

.simple-address--fixed-top {
  /* #ifndef APP-NVUE */
  display: flex;
  /* #endif */
  flex-direction: row;
  justify-content: space-between;
  border-top-color: $uni-border-color;
  border-top-style: solid;
  border-top-width: 1rpx;
}

.simple-address__header-btn-box {
  /* #ifndef APP-NVUE */
  display: flex;
  /* #endif */
  flex-direction: row;
  align-items: center;
  justify-content: center;
  height: 70rpx;
}

.simple-address__header-text {
  text-align: center;
  font-size: $uni-font-size-base;
  color: #1aad19;
  line-height: 70rpx;
  padding-left: 40rpx;
  padding-right: 40rpx;
}

.simple-address__box {
  position: relative;
}

.simple-address-view {
  position: relative;
  bottom: 0;
  left: 0;
  /* #ifndef APP-NVUE */
  width: 100%;
  /* #endif */
  /* #ifdef APP-NVUE */
  width: 750rpx;
  /* #endif */
  height: 408rpx;
  background-color: rgba(255, 255, 255, 1);
}

.picker-item {
  text-align: center;
  line-height: 70rpx;
  text-overflow: ellipsis;
  font-size: 28rpx;
}
</style>
