<template>
  <view class="tabbar">
    <view 
      class="tabbar-item" 
      :class="{ active: activeTab === 'home' }" 
      @click="switchTab('home')"
    >
      首页
    </view>
    <view 
      class="tabbar-item" 
      :class="{ active: activeTab === 'qa' }" 
      @click="switchTab('qa')"
    >
      直答
    </view>
    <view class="tabbar-item add-btn" @click="showAddMenu">
      <view class="add-circle">
        <image class="add-icon" src="/static/add.svg?v=1"></image>
      </view>
    </view>
    <view 
      class="tabbar-item" 
      :class="{ active: activeTab === 'message' }" 
      @click="switchTab('message')"
    >
      消息
    </view>
    <view 
      class="tabbar-item" 
      :class="{ active: activeTab === 'user' }" 
      @click="switchTab('user')"
    >
      我的
    </view>
  </view>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

// 定义属性
const props = defineProps({
  activeTab: {
    type: String,
    default: 'home'
  }
});

// 定义事件
const emit = defineEmits(['tab-click', 'add-click']);

// 切换标签页
const switchTab = (tab) => {
  if (tab === props.activeTab) return;
  emit('tab-click', tab);
  
  // 根据标签执行对应跳转
  switch (tab) {
    case 'home':
      navigateTo('/pages/index/index');
      break;
    case 'qa':
      // 直答页面暂未实现
      showToast('直答功能开发中');
      break;
    case 'message':
      navigateTo('/pages/message/index');
      break;
    case 'user':
      navigateTo('/pages/profile/index');
      break;
  }
};

// 显示添加菜单
const showAddMenu = () => {
  emit('add-click');
};

// 页面跳转封装
const navigateTo = (url) => {
  uni.navigateTo({
    url,
    fail: () => {
      // 尝试重定向方式跳转
      uni.redirectTo({
        url,
        fail: () => {
          // 如果还失败，可能是tabBar页面，使用switchTab
          uni.switchTab({
            url,
            fail: (err) => {
              console.error('导航失败:', err);
            }
          });
        }
      });
    }
  });
};

// 显示提示
const showToast = (title) => {
  uni.showToast({
    title,
    icon: 'none'
  });
};
</script>

<style lang="scss">
.tabbar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  display: flex;
  background-color: #fff;
  border-top: 1px solid #eee;
  box-shadow: 0 -2rpx 6rpx rgba(0, 0, 0, 0.02);
  z-index: 999;
  
  .tabbar-item {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 30rpx;
    color: #999;
    position: relative;
    height: 100%;
    
    &.active {
      color: #1677ff;
      font-weight: bold;
      
      &:after {
        content: '';
        position: absolute;
        bottom: 15rpx;
        left: 50%;
        transform: translateX(-50%);
        width: 40rpx;
        height: 4rpx;
        background-color: #1677ff;
      }
    }
  }
  
  .add-btn {
    position: relative;
    
    .add-circle {
      width: 80rpx;
      height: 80rpx;
      border-radius: 50%;
      background-color: #1677ff;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      display: flex;
      align-items: center;
      justify-content: center;
      box-shadow: 0 0 10rpx rgba(22, 119, 255, 0.4);
    }
    
    .add-icon {
      width: 40rpx;
      height: 40rpx;
    }
  }
}
</style> 