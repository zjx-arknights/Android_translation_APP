<template>
  <view class="splash-container" @click="handleClick">
    <view class="splash-text">BabelBreak</view>
  </view>
</template>

<script>
export default {
  methods: {
    handleClick() {
      // 点击后跳转到首页
      uni.redirectTo({
        url: '../home/home' // 确保路径正确
      });
    }
  }
}
</script>

<style>
.splash-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #000;
}

.splash-text {
  color: transparent;
  font-size: 60px;
  font-weight: bold;
  letter-spacing: 5px;
  background: linear-gradient(to right, #000, #fff, #000);
  -webkit-background-clip: text;
  background-clip: text;
  background-size: 80%;
  background-repeat: no-repeat;
  animation: move 2s linear infinite;
}

@keyframes move {
  0% { background-position: -500%; }
  100% { background-position: 500%; }
}
</style>