<template>
  <view class="icon-container">
    <image :src="iconSrc" :style="{ width: size + 'px', height: size + 'px', color: color }" />
  </view>
</template>

<script>
export default {
  props: {
    type: {
      type: String,
      required: true, // 图标类型是必传的
    },
    size: {
      type: [Number, String],
      default: 24, // 默认大小为 24px
    },
    color: {
      type: String,
      default: '#000', // 默认颜色为黑色
    }
  },
  computed: {
    iconSrc() {
      // 根据传入的 type 属性来选择相应的图标路径
      const icons = {
       reload: '/static/reload.svg',
	   zh: '/static/zh.svg',
	   en: '/static/en.svg'
        // 添加更多的图标名称和路径
      };
      // 如果传入的 type 对应的图标不存在，返回默认的图标
      return icons[this.type] || '/static/icons/default.png';
    }
  }
}
</script>

<style scoped>
.icon-container {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
