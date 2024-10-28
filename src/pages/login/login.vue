<template>
  <view class="viewport">
    <view class="logo">
      <image src="../../static/log.png"></image>
    </view>
    <view class="login">
      <button class="button wechat" @tap="weChatLogin">
        <text class="icon icon-wechat"></text>
        微信快捷登录
      </button>
      <!-- <label class="rempass">
        <checkbox value="remPassword" :checked="remPassword" @click="remPasswordChange" color="#06afe3"
          style="transform: scale(0.7)" />记住密码
      </label> -->
      <view class="tips">
        <view class="signUp-tip-txt" :class="'animation-' + animation">
          <checkbox class="protocol-radio" :checked="agree" @click="agreeChange" color="#3964e6" style="
              transform: scale(0.55) translate(45px, 0px);
              width: 73px;
              height: 73px;
              line-height: 73px;
              margin-left: -14%;
            ">
          </checkbox>
          同意
          <text class="signUp-tip-txt-protocol">《用户协议》</text>
          和
          <text class="signUp-tip-txt-protocol">《隐私政策》</text>
        </view>
      </view>
      <!-- 用用户名密码登录 -->
      <text class="centered-text" @tap="userLogin">用户名密码登录</text>
    </view>
  </view>
</template>

<script setup>
import { Base64 } from 'js-base64'
import randomWord from '@/utils/random'
import { onLoad } from '@dcloudio/uni-app'
import { ref } from 'vue'
import { logInAPI } from '@/services/logIn'
import { baseUrl } from '@/utils/http'

// 获取code登陆凭证
let code = ''

// 微信快捷登录
const weChatLogin = async () => {
  if (agree.value) {
    const res = await wx.login()
    console.log(res)
    code = res.code
    if (res) {
      uni.showToast({
        title: '获取登录code成功',
        icon: 'success',
        duration: 1000,
      })
      // 调用微信登录API
      wx.request({
        url: `${baseUrl}/wxLogin`, //改基地址的时候记得改这里
        method: 'POST',
        data: {
          code: code,
        },
        success: (res) => {
          if (res.data.code === 200) {
            // 存储token
            uni.setStorageSync('wxToken', res.data.wxToken)
            uni.showToast({
              title: '微信快捷登录成功',
              icon: 'success',
              duration: 1000,
            })
            if (res.data.bind === true) {
              // setTimeout(() => {
              //   uni.navigateTo({
              //     url: '/pages/login/bind',
              //   })
              // }, 2000)
              // uni.showToast({
              //   title: '请绑定学号账号',
              //   icon: 'none',
              //   duration: 1000,
              // })
              uni.showToast({
                title: '登录成功',
                icon: 'success',
                duration: 1000,
              })
              uni.setStorageSync('token', res.data.wxToken)
              setTimeout(() => {
                uni.switchTab({
                  url: '/pages/index/index',
                })
              }, 2000)
            } else {
              setTimeout(() => {
                uni.navigateTo({
                  url: '/pages/login/bind',
                })
              }, 2000)
              uni.showToast({
                title: '请绑定学号账号',
                icon: 'none',
                duration: 1000,
              })
            }
          } else {
            uni.showToast({
              title: '登录失败',
              icon: 'error',
              duration: 1000,
            })
          }
        },
        fail: () => {
          uni.showToast({
            title: '请求失败',
            icon: 'error',
            duration: 1000,
          })
        },
      })
    } else {
      uni.showToast({
        title: '获取登录code失败',
        icon: 'error',
        duration: 1000,
      })
    }
  } else {
    shakeAnimation()
  }
}

const userLogin = () => {
  uni.navigateTo({
    url: '/pages/login/userLogin',
  })
}


//隐私政策
const agree = ref(false)
const animation = ref('')
//勾选同意协议
const agreeChange = () => {
  agree.value = !agree.value
}
const shakeAnimation = () => {
  console.log('抖动')
  uni.showToast({ title: '请您勾选同意协议', icon: 'none' })

  animation.value = 'shake'
  setTimeout(() => {
    animation.value = ''
  }, 900)
}
//记住密码
const remPassword = ref(false)
const remPasswordChange = () => {
  remPassword.value = !remPassword.value
}
onLoad(() => {
  const token = uni.getStorageSync("token")
  if (token) {
    uni.switchTab({
      url: '/pages/index/index',
    })
  } else {
    console.log('未登录')
  }
})
</script>



<style lang="scss">
page {
  height: 100%;
}

.viewport {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 20rpx 40rpx;
}

.logo {
  flex: 1;
  text-align: center;
  margin-bottom: 60rpx;

  image {
    width: 220rpx;
    height: 220rpx;
    margin-top: 10vh;
  }
}

.login {
  width: 90%;
  display: flex;
  flex-direction: column;
  height: 60vh;
  padding: 40rpx 20rpx 20rpx;

  .button {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 80rpx;
    font-size: 28rpx;
    border-radius: 72rpx;
    color: #fff;

    .icon {
      font-size: 40rpx;
      margin-right: 6rpx;
    }
  }

  .rempass {
    display: block;
    margin: 20rpx auto;
    font-size: 24rpx;
    color: #444;
  }

  .wechat {
    background-color: #34a9d6;

    .options {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-top: 70rpx;

      button {
        padding: 0;
        background-color: transparent;
      }
    }

    .icon {
      font-size: 24rpx;
      color: #444;
      display: flex;
      flex-direction: column;
      align-items: center;

      &::before {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 80rpx;
        height: 80rpx;
        margin-bottom: 6rpx;
        font-size: 40rpx;
        border: 1rpx solid #444;
        border-radius: 50%;
      }
    }

    .icon-weixin::before {
      border-color: #06c05f;
      color: #06c05f;
    }
  }
  .centered-text {
    text-align: center;
    width: 100%;
    display: block;
    margin-top: 20rpx;
    font-size: 28rpx;
    color: #444;
    text-decoration: underline;
  }
}

.tips {
  // position: absolute;
  // bottom: 80rpx;
  // left: 20rpx;
  // right: 20rpx;
  font-size: 23rpx;
  color: #999;
  text-align: center;
}

.signUp-tip-txt-protocol {
  color: red;
}

//勾选协议 抖动动画
.animation-shake {
  animation-delay: 0s;
  animation-duration: 0.1s;
  animation-iteration-count: infinite;
  animation-name: shake;
  animation-play-state: running;
  animation-timing-function: ease-in-out;
}

@keyframes shake {
  0% {
    transform: translate(0, 0) rotate(0deg);
  }

  2% {
    transform: translate(-4px, 0) rotate(0deg);
  }

  4% {
    transform: translate(2px, 0) rotate(0deg);
  }

  6% {
    transform: translate(-3px, 0) rotate(0deg);
  }

  8% {
    transform: translate(9px, 0) rotate(0deg);
  }

  10% {
    transform: translate(2px, 0) rotate(0deg);
  }

  12% {
    transform: translate(-6px, 0) rotate(0deg);
  }

  14% {
    transform: translate(-7px, 0) rotate(0deg);
  }

  16% {
    transform: translate(2px, 0) rotate(0deg);
  }

  18% {
    transform: translate(7px, 0) rotate(0deg);
  }

  20% {
    transform: translate(-1px, 0) rotate(0deg);
  }

  22% {
    transform: translate(-10px, 0) rotate(0deg);
  }

  24% {
    transform: translate(-10px, 0) rotate(0deg);
  }

  26% {
    transform: translate(3px, 0) rotate(0deg);
  }

  28% {
    transform: translate(-5px, 0) rotate(0deg);
  }

  30% {
    transform: translate(4px, 0) rotate(0deg);
  }

  32% {
    transform: translate(-6px, 0) rotate(0deg);
  }

  34% {
    transform: translate(-2px, 0) rotate(0deg);
  }

  36% {
    transform: translate(-6px, 0) rotate(0deg);
  }

  38% {
    transform: translate(-2px, 0) rotate(0deg);
  }

  40% {
    transform: translate(4px, 0) rotate(0deg);
  }

  42% {
    transform: translate(8px, 0) rotate(0deg);
  }

  44% {
    transform: translate(-3px, 0) rotate(0deg);
  }

  46% {
    transform: translate(-10px, 0) rotate(0deg);
  }

  48% {
    transform: translate(4px, 0) rotate(0deg);
  }

  50% {
    transform: translate(-6px, 0) rotate(0deg);
  }

  52% {
    transform: translate(6px, 0) rotate(0deg);
  }

  54% {
    transform: translate(-8px, 0) rotate(0deg);
  }

  56% {
    transform: translate(5px, 0) rotate(0deg);
  }

  58% {
    transform: translate(9px, 0) rotate(0deg);
  }

  60% {
    transform: translate(7px, 0) rotate(0deg);
  }

  62% {
    transform: translate(1px, 0) rotate(0deg);
  }

  64% {
    transform: translate(2px, 0) rotate(0deg);
  }

  66% {
    transform: translate(-2px, 0) rotate(0deg);
  }

  68% {
    transform: translate(-7px, 0) rotate(0deg);
  }

  70% {
    transform: translate(-1px, 0) rotate(0deg);
  }

  72% {
    transform: translate(-6px, 0) rotate(0deg);
  }

  74% {
    transform: translate(5px, 0) rotate(0deg);
  }

  76% {
    transform: translate(0, 0) rotate(0deg);
  }

  78% {
    transform: translate(9px, 0) rotate(0deg);
  }

  80% {
    transform: translate(-3px, 0) rotate(0deg);
  }

  82% {
    transform: translate(8px, 0) rotate(0deg);
  }

  84% {
    transform: translate(-2px, 0) rotate(0deg);
  }

  86% {
    transform: translate(-1px, 0) rotate(0deg);
  }

  88% {
    transform: translate(-3px, 0) rotate(0deg);
  }

  90% {
    transform: translate(4px, 0) rotate(0deg);
  }

  92% {
    transform: translate(0, 0) rotate(0deg);
  }

  94% {
    transform: translate(4px, 0) rotate(0deg);
  }

  96% {
    transform: translate(2px, 0) rotate(0deg);
  }

  98% {
    transform: translate(-2px, 0) rotate(0deg);
  }
}
</style>