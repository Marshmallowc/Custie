<script>
import { store } from './common/store.js';

export default {
  onLaunch: async function () {
    console.log('App Launch');
    
    // 初始化全局数据
    try {
      await store.init();
      console.log('全局数据初始化完成');
    } catch (error) {
      console.error('全局数据初始化失败', error);
    }
  },
  onShow: function () {
    console.log('App Show');
  },
  onHide: function () {
    console.log('App Hide');
  },
}
</script>

<style lang="scss">
/* 全局样式 */
page {
  font-family: -apple-system, BlinkMacSystemFont, 'Helvetica Neue', Helvetica,
    Segoe UI, Arial, Roboto, 'PingFang SC', 'miui', 'Hiragino Sans GB', 'Microsoft Yahei',
    sans-serif;
  box-sizing: border-box;
}

/* 清除默认样式 */
view, text, input, button, textarea {
  box-sizing: border-box;
}

/* 主色调 */
.primary-color {
  color: #1677ff;
}

.primary-bg {
  background-color: #1677ff;
}

/* 文本颜色 */
.text-primary {
  color: #333333;
}

.text-secondary {
  color: #666666;
}

.text-muted {
  color: #999999;
}

/* 通用边距 */
.page-container {
  padding: 30rpx;
}

/* 阴影 */
.card-shadow {
  box-shadow: 0 2rpx 12rpx 0 rgba(0, 0, 0, 0.1);
}

/* 圆角 */
.rounded {
  border-radius: 8rpx;
}

.rounded-lg {
  border-radius: 16rpx;
}

.rounded-full {
  border-radius: 999rpx;
}

/* 按钮样式 */
.btn {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20rpx 30rpx;
  border-radius: 45rpx;
  font-size: 28rpx;
  font-weight: bold;
}

.btn-primary {
  background-color: #1677ff;
  color: #ffffff;
}

.btn-outline {
  border: 2rpx solid #1677ff;
  color: #1677ff;
  background-color: transparent;
}
</style>
