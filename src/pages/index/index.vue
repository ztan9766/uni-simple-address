<template>
  <view class="content">
    <view @tap="onAddressOpen">
      {{ address.label || '您的城市' }}
    </view>

    <simple-address ref="simpleAddress" :pickerValueDefault="addressIndex" @onConfirm="onAddressConfirm">
    </simple-address>
  </view>
</template>

<script>
import SimpleAddress from '../../../simple-address/simple-address'

export default {
  name: 'Index',
  components: {
    SimpleAddress,
  },
  data() {
    return {
      title: 'Hello',
      address: {},
      addressIndex: [0, 0, 0],
    }
  },
  onLoad() {},
  methods: {
    onAddressOpen() {
      this.$refs.simpleAddress.open()
    },
    onAddressConfirm(e) {
      console.log(e)
      this.address = e
    },
  },
}
</script>

<style>
.content {
  padding: 30rpx;
  margin: 30rpx;
  text-align: center;
  border: 1px solid;
  border-radius: 8rpx;
}
</style>
