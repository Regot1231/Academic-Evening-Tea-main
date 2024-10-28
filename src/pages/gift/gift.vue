<template>
  <div :style="{ paddingBottom: safeAreaInsets?.bottom!+ 0 + 'px' }">
    <view class="ad_popError" v-if="mycredit == 0">您的积分为零</view>
    <!-- <swiper
      :indicator-dots="indicatorDots"
      :autoplay="autoplay"
      :interval="interval"
      :duration="duration"
    > -->
    <!-- <swiper-item v-for="(url, index) in goodsdata.imgs" :key="index"> :data-src="url" @click="previewImage" -->
    <view class="goodsimg">
      <image :src="goodsdata.coverPicture" mode="heightFix" />
    </view>

    <!-- </swiper-item> -->
    <!-- </swiper> -->

    <scroll-view :scroll-y="true">
      <view class="detail">
        <text class="price">{{ goodsdata.price }}积分</text>
        <view class="title-view">
          <text class="title">{{ goodsdata.name }}</text>
          <uni-tag
            :text="`总库存：${goodsdata.allnums}`"
            size="mini"
            :inverted="true"
            type="success"
          />
          <!-- <text >总库存：{{ goodsdata.allnums }}</text> -->
        </view>
        <text class="text-remark">{{ goodsdata.introduce }}</text>
      </view>

      <view class="separate"></view>
      <!-- sku选择 -->
      <view class="num-box">
        <text bindtap="toggleDialog">请选择兑换数量</text>
        <uni-number-box
          class="numchooseBox"
          @blur="blur"
          :max="maxNum"
          v-model="goodsnum"
        ></uni-number-box>
      </view>

      <view class="separate"></view>
      <image
        mode="widthFix"
        class="goodsdimg"
        v-for="(url, index) in goodsdata.imgs"
        :key="index"
        :src="url"
        @click="previewImage"
      />
      <view class="temp"></view>
    </scroll-view>

    <view class="detail-nav" :style="{ bottom: safeAreaInsets?.bottom!+ 0 + 'px' }">
      <view class="line_nav" :style="{ height: safeAreaInsets?.bottom!+ 1 + 'px' }"></view>
      <view class="score-box">
        <view class="scorenum">所需积分：</view>
        <image
          style="border-radius: 20rpx; width: 40rpx; height: 40rpx; margin-top: 10rpx"
          src="../../static/gifts/R-C.jpg"
        />
        <view class="scorenum">{{ scoreres }}</view>
      </view>
      <!-- <button class="button-green" bindtap="addCar" formType="submit">加入购物车</button> -->
      <button :disabled="goodsnum == 0" class="button-red" @click="immeBuy" formType="submit">
        立即兑换
      </button>
    </view>
  </div>
</template>

<script setup lang="ts">
// import { ref, reactive } from 'vue'
// import { onLoad } from '@dcloudio/uni-app';

// onLoad(() => {
//   uni.showToast({
//     title: '礼品中心部分功能尚未实现，敬请期待~',
//     icon: 'none',
//     duration: 2000,
//     complete: function () {
//       setTimeout(function () {
//         uni.switchTab({
//           url: '/pages/index/index'
//         });
//       }, 2000); // 在弹窗关闭后，延迟2秒执行页面跳转
//     }
//   })
// })

import { onLoad, onShow } from '@dcloudio/uni-app'
import { ref, computed, watch } from 'vue'
import { getGiftDetail, exchangeGift } from '@/services/gift'
import { getPersonalInf } from '@/services/personalInf'
// 获取屏幕边界到安全区域距离
const { safeAreaInsets } = uni.getSystemInfoSync()

const props = defineProps<{
  id: number
}>()

const getGiftDetails = async (id) => {
  const res = await getGiftDetail(id)
  console.log(res.data)
  goodsdata.value = res.data
}
const getMyCredit = async () => {
  const res = await getPersonalInf()
  console.log(res.data.credit)
  mycredit.value = res.data.credit
}
const mycredit = ref<number>(1)
const blur = (e) => {
  console.log(e)
  if (e.detail.value > goodsdata.value.allnums) {
    uni.showToast({
      title: '兑换数量超出库存',
      icon: 'none',
      duration: 1000,
    })
  }
}
const maxNum = computed(() => {
  if (mycredit.value == 0) {
    return 0
  }
  return goodsdata.value.allnums
})
const scoreres = computed(() => {
  return (goodsnum.value * goodsdata.value.price).toFixed(2)
})
// watch([goodsnum.value, () => goodsdata.value.price], ([newgoodsnum]) => {
//   scoreres.value = newgoodsnum * goodsdata.value.price;
// });
const goodsdata = ref({})
const goodsnum = ref<number>(0)
onShow(() => {
  if (uni.getStorageSync('token')) {
    getMyCredit()
  }
})
onLoad(() => {
  if (uni.getStorageSync('token')) {
    console.log(props.id)
    getGiftDetails(props.id)
    getMyCredit()
  } else {
    wx.showToast({
      title: '您尚未登录，请先登录',
      icon: 'none',
      duration: 1000,
      complete: () => {
        setTimeout(() => {
          uni.navigateTo({
            url: '/pages/login/login',
          })
        }, 1000)
      },
    })
  }
})

const indicatorDots = true
const autoplay = true
const interval = 3000
const duration = 1000

const previewImage = (e) => {
  var current = e.target.dataset.src
  console.log(current)
  wx.previewImage({
    current: current,
    urls: goodsdata.value.imgs,
  })
}

const applyExchange = async (data) => {
  const res = await exchangeGift(data)
  console.log(res)
  if (res.code == 200) {
    if (res.data == 3) {
      uni.showToast({
        title: '兑换成功！',
        icon: 'success',
        duration: 2000,
        complete: function () {
          setTimeout(function () {
            getGiftDetails(props.id)
            getMyCredit()
            goodsnum.value = 0
            // uni.navigateTo({
            //   url: '/pages/gift/giftCenter',
            // })
          }, 2000) // 在弹窗关闭后，延迟2秒执行页面跳转
        },
      })
    }
    if (res.data == 2) {
      uni.showToast({
        title: '余额不足！',
        icon: 'error',
        duration: 2000,
      })
    }
    if (res.data == 1) {
      uni.showToast({
        title: '库存不足！',
        icon: 'error',
        duration: 2000,
      })
    }
  }
}
const immeBuy = () => {
  console.log('购买')
  if (mycredit.value < goodsnum.value * goodsdata.value.price) {
    uni.showToast({
      title: '剩余积分不足',
      icon: 'error',
      duration: 1000,
    })
    return
  }
  wx.showModal({
    title: '确认兑换',
    content: '确定要兑换吗？',
    success(res) {
      if (res.confirm) {
        //用户点击了确认按钮
        applyExchange({
          id: null,
          goodsId: goodsdata.value.id,
          nums: goodsnum.value,
        })
      } else if (res.cancel) {
        // 用户点击了取消按钮
        console.log('用户取消了兑换')
      }
    },
  })
}
</script>
<style scoped>
page {
  display: flex;
  flex-direction: column;
  height: 100%;
}
/* 直接设置swiper属性 */
swiper {
  height: 500rpx;
}
swiper-item image {
  width: 100%;
  height: 100%;
}
.separate {
  height: 20rpx;
  background-color: whitesmoke;
  margin-top: 10rpx;
  width: 100%;
}
.ad_popError {
  background: #de352d;
  color: #fff;
  height: 58rpx;
  line-height: 58rpx;
  font-size: 24rpx;
  text-align: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 3;
}
.goodsimg {
  height: 600rpx;
  width: 100%;
  display: flex;
  justify-content: space-around;
}
.detail {
  display: flex;
  flex-direction: column;
  margin: 15rpx 10rpx 0;
}
.detail .title {
  font-size: 40rpx;
  color: black;
  text-align: justify;
}
.detail .price {
  color: red;
  font-size: 40rpx;
  margin: 10rpx;
  margin-top: 30rpx;
}
.title-view {
  display: flex;
  justify-content: space-between;
}
.line_flag {
  width: 80rpx;
  height: 1rpx;
  display: inline-block;
  margin: 20rpx auto;
  background-color: gainsboro;
  text-align: center;
}
.line {
  width: 100%;
  height: 2rpx;
  display: inline-block;
  margin: 20rpx 0rpx;
  background-color: gainsboro;
  text-align: center;
}
.detail-nav {
  display: flex;
  flex-direction: row;
  align-items: center;
  float: left;
  background-color: #fff;
  position: fixed;
  bottom: 0;
  right: 0;
  z-index: 1;
  width: 100%;
  height: 100rpx;
}
.goodsdimg {
  margin-bottom: 7px;
}
.button-red {
  background-color: #f44336; /* 红色 */
}
.image_detail {
  width: 100%;
}
button {
  z-index: 999;
  color: white;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 30rpx;
  border-radius: 0rpx;
  width: 50%;
  height: 100%;
  line-height: 100rpx;
}
.detail-nav image {
  width: 70rpx;
  height: 50rpx;
  margin: 20rpx 40rpx;
}
.line_nav {
  width: 100%;
  background-color: #fff;
  position: fixed;
  bottom: 0;
}
/* 占位 */
.temp {
  height: 100rpx;
}
text {
  display: block;
  /* height: 50rpx; */
  line-height: 50rpx;
  /* font-size: 30rpx; */
  margin: 10rpx;
}
.text-remark {
  display: block;
  font-size: 27rpx;
}
.num-box {
  font-size: 27rpx;
  display: flex;
  flex-direction: row;
  justify-content: center;
}
.score-box {
  display: flex;
  flex-direction: row;
}
.scorenum {
  color: #f44336;
  display: block;
  font-size: 30rpx;
  margin: 10rpx;
}
.numchooseBox {
  margin: auto 0;
}
</style>
