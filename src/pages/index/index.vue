<template>
  <view class="index-container">
    <!-- 头部搜索栏 -->
    <view class="search-header">
      <view class="search-box">
        <input type="text" placeholder="搜索" />
        <view class="search-btn">搜索</view>
      </view>
    </view>
    
    <!-- 标签切换栏 -->
    <view class="tab-bar">
      <view class="tab-item" :class="{ active: activeTab === 'all' }" @click="changeTab('all')">
        <text>所有</text>
      </view>
      <view class="tab-item" :class="{ active: activeTab === 'follow' }" @click="changeTab('follow')">
        <text>关注</text>
      </view>
      <view class="tab-item" :class="{ active: activeTab === 'recommend' }" @click="changeTab('recommend')">
        <text>推荐</text>
      </view>
      <view class="tab-item" :class="{ active: activeTab === 'hot' }" @click="changeTab('hot')">
        <text>热榜</text>
      </view>
    </view>
    
    <!-- 问题列表 -->
    <view class="question-list">
      <!-- 加载中提示 -->
      <view class="loading-container" v-if="loading && questionList.length === 0">
        <view class="loading-spinner"></view>
        <text class="loading-text">加载中...</text>
      </view>
      
      <!-- 无数据提示 -->
      <view class="empty-container" v-else-if="questionList.length === 0">
        <text class="empty-text">暂无数据</text>
      </view>
      
      <!-- 问题项 -->
      <view class="question-item" v-for="(item, index) in questionList" :key="index" @click="goToArticle(item)">
        <!-- 问题标题 -->
        <view class="question-title">{{ item.title }}</view>
        
        <!-- 问题信息 -->
        <view class="question-info">
          <!-- 用户信息 -->
          <view class="user-info">
            <img class="user-avatar" :src="item.avatar || '/static/avatar/user1.png'" />
            <text class="user-name">{{ item.userName || '匿名用户' }}</text>
          </view>
          
          <!-- 问题描述 -->
          <view class="question-description">{{ item.description || '暂无描述' }}</view>
        </view>
        
        <!-- 底部统计 -->
        <view class="question-footer">
          <text class="time">{{ formatDateTime(item.time || item.createdAt) }}</text>
          <view class="stats">
            <view class="stat-item">
              <text class="count">{{ item.viewCount || 0 }}</text>
              <text class="label">浏览</text>
            </view>
            <view class="stat-item">
              <text class="count">{{ item.answerCount || 0 }}</text>
              <text class="label">回答</text>
            </view>
          </view>
        </view>
      </view>
      
      <!-- 加载更多提示 -->
      <view class="loading-more" v-if="loading && questionList.length > 0">
        <view class="loading-spinner"></view>
        <text class="loading-text">加载更多...</text>
      </view>
      
      <!-- 无更多数据提示 -->
      <view class="no-more" v-if="!loading && !pagination.hasMore && questionList.length > 0">
        <text class="no-more-text">— 已经到底了 —</text>
      </view>
    </view>
    
    <!-- 底部导航栏 -->
    <TabBar active-tab="home" @add-click="toggleActionMenu" />
    
    <!-- 底部弹出菜单 -->
    <view class="action-menu" v-if="showActionMenu">
      <view class="action-menu-content" @click.stop>
        <view class="action-menu-header">
          <text class="header-title">请选择操作</text>
          <view class="close-btn" @click="toggleActionMenu">×</view>
        </view>
        <view class="action-menu-items">
          <view class="action-menu-item" @click="handleAsk">
            <view class="action-icon-wrapper ask-icon">
              <image class="action-icon" src="/static/ask.svg"></image>
            </view>
            <text class="action-text">提问</text>
          </view>
          <view class="action-menu-item" @click="handleAnswer">
            <view class="action-icon-wrapper answer-icon">
              <image class="action-icon" src="/static/answer.svg"></image>
            </view>
            <text class="action-text">回答问题</text>
          </view>
        </view>
      </view>
    </view>
    
    <!-- 遮罩层 -->
    <view class="mask" v-if="showActionMenu" @click="toggleActionMenu"></view>
  </view>
</template>

<script setup>
import { ref, onMounted, reactive } from 'vue';
import TabBar from '@/components/TabBar.vue';
import { store } from '@/common/store.js';
import { onPullDownRefresh, onReachBottom } from '@dcloudio/uni-app';

// 当前激活的标签
const activeTab = ref('all');

// 是否显示底部操作菜单
const showActionMenu = ref(false);

// 问题列表数据
const questionList = ref([]);

// 分页参数
const pagination = reactive({
  page: 1,
  limit: 10,
  total: 0,
  hasMore: true
});

// 加载状态
const loading = ref(false);

// 切换标签
const changeTab = (tab) => {
  // 重置分页
  pagination.page = 1;
  pagination.hasMore = true;
  
  activeTab.value = tab;
  loadQuestionList(tab, true);
};

// 获取标签名称
const getTabName = (tab) => {
  switch (tab) {
    case 'follow':
      return '关注';
    case 'recommend':
      return '推荐';
    case 'hot':
      return '热榜';
    default:
      return '';
  }
};

// 切换底部操作菜单显示状态
const toggleActionMenu = () => {
  showActionMenu.value = !showActionMenu.value;
};

// 处理提问操作
const handleAsk = () => {
  showActionMenu.value = false;
  // 跳转到提问页面
  uni.navigateTo({
    url: '/pages/ask/index'
  });
};

// 处理回答操作
const handleAnswer = () => {
  showActionMenu.value = false;
  // 跳转到回答页面
  uni.navigateTo({
    url: '/pages/answer/index'
  });
};

// 加载问题列表
const loadQuestionList = async (tab, isRefresh = false) => {
  if (loading.value) return;
  
  // 如果是刷新，重置页码
  if (isRefresh) {
    pagination.page = 1;
    pagination.hasMore = true;
  }
  
  // 如果没有更多数据，不再请求
  if (!pagination.hasMore && !isRefresh) return;
  
  loading.value = true;
  try {
    // 调用API获取问题列表
    const params = {
      tab: tab,
      page: pagination.page,
      limit: pagination.limit
    };
    
    const response = await store.fetchQuestions(params);
    
    // 调试输出
    console.log('问题列表数据样例:', response.slice(0, 2).map(q => ({
      title: q.title,
      userName: q.userName,
      avatar: q.avatar
    })));
    
    // 更新问题列表
    if (isRefresh) {
      // 直接使用处理好的数据
      questionList.value = response || [];
    } else {
      // 直接使用处理好的数据
      questionList.value = [...questionList.value, ...(response || [])];
    }
    
    // 更新分页信息
    pagination.total = store.questions.length > 0 ? store.questions.length : 0;
    pagination.hasMore = questionList.value.length < pagination.total;
    
    // 成功后增加页码
    if (questionList.value.length > 0) {
      pagination.page++;
    }
  } catch (error) {
    console.error('加载问题列表失败:', error);
    uni.showToast({
      title: '加载失败，请重试',
      icon: 'none'
    });
  } finally {
    loading.value = false;
    // 加载完成，停止下拉刷新
    uni.stopPullDownRefresh();
  }
};

// 下拉刷新处理函数
onPullDownRefresh(() => {
  loadQuestionList(activeTab.value, true);
});

// 上拉加载更多处理函数
onReachBottom(() => {
  loadQuestionList(activeTab.value);
});

// 添加点击问题项跳转到文章详情页的功能
const goToArticle = (item) => {
  // 获取问题ID，支持不同的属性名
  let questionId = null;
  
  // 优先使用字符串形式的ID
  if (item._id && typeof item._id === 'string') {
    questionId = item._id;
  } else if (item.id && typeof item.id === 'string') {
    questionId = item.id;
  } else if (item._id && typeof item._id === 'object' && item._id.toString) {
    // 对象形式的ID转为字符串
    questionId = item._id.toString();
  } else if (item.id && typeof item.id !== 'string') {
    // 非字符串ID转为字符串
    questionId = String(item.id);
  } else if (item._id) {
    // 其他情况，尝试转字符串
    questionId = String(item._id);
  } else if (item.id) {
    questionId = String(item.id);
  }
  
  // 输出调试信息
  console.log('跳转问题详情，问题数据:', JSON.stringify({
    title: item.title,
    _id: item._id,
    id: item.id,
    finalId: questionId,
    idType: typeof questionId
  }));
  
  // 确保有有效的ID
  if (!questionId) {
    uni.showToast({
      title: '无效的文章ID',
      icon: 'none'
    });
    return;
  }
  
  // 确保ID是字符串类型
  const idStr = String(questionId);
  
  // 跳转到文章详情页，使用简单字符串参数
  uni.navigateTo({
    url: `/pages/article/index?id=${idStr}`
  });
};

// 格式化日期时间
const formatDateTime = (dateTimeStr) => {
  if (!dateTimeStr) return '未知时间';
  
  // 检查是否是相对时间格式（如"3天前"）
  if (typeof dateTimeStr === 'string' && 
      (dateTimeStr.includes('天前') || 
       dateTimeStr.includes('小时前') || 
       dateTimeStr.includes('分钟前') || 
       dateTimeStr.includes('周前'))) {
    return dateTimeStr;
  }
  
  try {
  const date = new Date(dateTimeStr);
  
  // 检查日期是否有效
  if (isNaN(date.getTime())) {
    return dateTimeStr;
  }
  
    // 计算相对时间
    const now = new Date();
    const diffMs = now - date;
    const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
    
    if (diffDays === 0) {
      const diffHours = Math.floor(diffMs / (1000 * 60 * 60));
      if (diffHours === 0) {
        const diffMinutes = Math.floor(diffMs / (1000 * 60));
        if (diffMinutes === 0) {
          return '刚刚';
        }
        return `${diffMinutes}分钟前`;
      }
      return `${diffHours}小时前`;
    } else if (diffDays < 7) {
      return `${diffDays}天前`;
    } else if (diffDays < 30) {
      return `${Math.floor(diffDays / 7)}周前`;
    } else {
      return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;
    }
  } catch (error) {
    return dateTimeStr;
  }
};

// 页面加载时执行
onMounted(() => {
  loadQuestionList(activeTab.value, true);
});
</script>

<style lang="scss">
.index-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f5f5f5;
  position: relative;
  padding-bottom: 100rpx; // 为底部导航留出空间
}

// 头部搜索栏
.search-header {
  padding: 20rpx 30rpx;
  background-color: #fff;
  
  .search-box {
    display: flex;
    height: 80rpx;
    border: 1px solid #eee;
    border-radius: 40rpx;
    overflow: hidden;
    
    input {
      flex: 1;
      height: 80rpx;
      padding: 0 30rpx;
      font-size: 28rpx;
      border: none;
    }
    
    .search-btn {
      width: 120rpx;
      height: 80rpx;
      background: #fff;
      font-size: 28rpx;
      color: #333;
      line-height: 80rpx;
      text-align: center;
      border-left: 1px solid #eee;
    }
  }
}

// 标签切换栏
.tab-bar {
  display: flex;
  height: 80rpx;
  background-color: #fff;
  border-bottom: 1px solid #f0f0f0;
  
  .tab-item {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 28rpx;
    color: #666;
    position: relative;
    
    &.active {
      color: #333;
      font-weight: bold;
      
      &:after {
        content: '';
        position: absolute;
        bottom: 0;
        left: 50%;
        transform: translateX(-50%);
        width: 40rpx;
        height: 4rpx;
        background-color: #1677ff;
        border-radius: 2rpx;
      }
    }
  }
}

// 问题列表
.question-list {
  flex: 1;
  padding: 20rpx 30rpx;
  
  .question-item {
    margin-bottom: 20rpx;
    background-color: #fff;
    border-radius: 12rpx;
    padding: 30rpx;
    box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.05);
    
    .question-title {
      font-size: 32rpx;
      font-weight: bold;
      color: #333;
      margin-bottom: 20rpx;
      line-height: 1.4;
    }
    
    .question-info {
      display: flex;
      flex-direction: column;
      margin-bottom: 20rpx;
      
      .user-info {
        display: flex;
        align-items: center;
        margin-bottom: 15rpx;
        
        .user-avatar {
          width: 60rpx;
          height: 60rpx;
          border-radius: 50%;
          margin-right: 12rpx;
          border: 1px solid #f0f0f0;
          object-fit: cover;
        }
        
        .user-name {
          font-size: 26rpx;
          color: #333;
          font-weight: 500;
        }
      }
      
      .question-description {
        font-size: 28rpx;
        color: #666;
        line-height: 1.5;
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
      }
    }
    
    .question-footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      
      .time {
        font-size: 24rpx;
        color: #999;
      }
      
      .stats {
        display: flex;
        
        .stat-item {
          display: flex;
          align-items: center;
          margin-left: 20rpx;
          
          .count {
            font-size: 24rpx;
            color: #666;
            margin-right: 4rpx;
          }
          
          .label {
            font-size: 24rpx;
            color: #999;
          }
        }
      }
    }
  }
  
  // 加载中状态
  .loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 40rpx;
    
    .loading-spinner {
      width: 60rpx;
      height: 60rpx;
      border: 4rpx solid #f3f3f3;
      border-top: 4rpx solid #1677ff;
      border-radius: 50%;
      animation: spin 1s linear infinite;
      margin-bottom: 20rpx;
    }
    
    .loading-text {
      font-size: 28rpx;
      color: #999;
    }
  }
  
  // 空数据状态
  .empty-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 100rpx 0;
    
    .empty-text {
      font-size: 28rpx;
      color: #999;
    }
  }
  
  // 加载更多
  .loading-more {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 30rpx 0;
    
    .loading-spinner {
      width: 40rpx;
      height: 40rpx;
      border: 3rpx solid #f3f3f3;
      border-top: 3rpx solid #1677ff;
      border-radius: 50%;
      animation: spin 1s linear infinite;
      margin-right: 10rpx;
    }
    
    .loading-text {
      font-size: 24rpx;
      color: #999;
    }
  }
  
  // 无更多数据
  .no-more {
    text-align: center;
    padding: 30rpx 0;
    
    .no-more-text {
      font-size: 24rpx;
      color: #999;
    }
  }
}

// 底部操作菜单
.action-menu {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  
  .action-menu-content {
    width: 100%;
    background-color: #fff;
    border-top-left-radius: 20rpx;
    border-top-right-radius: 20rpx;
    padding: 40rpx 30rpx 60rpx;
    display: flex;
    flex-direction: column;
    align-items: center;
    animation: slideUp 0.3s ease;
    box-shadow: 0 -4rpx 20rpx rgba(0, 0, 0, 0.05);
    
    .action-menu-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      padding: 0 20rpx 30rpx;
      border-bottom: 1px solid #f5f5f5;
      margin-bottom: 30rpx;
      
      .header-title {
        font-size: 28rpx;
        color: #666;
      }
      
      .close-btn {
        width: 40rpx;
        height: 40rpx;
        font-size: 32rpx;
        color: #999;
        display: flex;
  align-items: center;
  justify-content: center;
}
    }
    
    .action-menu-items {
      display: flex;
      justify-content: space-around;
      width: 100%;
      padding: 0 20rpx;
      
      .action-menu-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 20rpx 40rpx;
        border-radius: 12rpx;
        transition: all 0.2s;
        
        &:active {
          background-color: #f9f9f9;
        }
        
        .action-icon-wrapper {
          width: 110rpx;
          height: 110rpx;
          border-radius: 50%;
  display: flex;
          align-items: center;
  justify-content: center;
          margin-bottom: 20rpx;
          transition: all 0.2s;
          
          &.ask-icon {
            background-color: #e8f4ff;
            border: 1px solid #d0e8ff;
          }
          
          &.answer-icon {
            background-color: #e8fff0;
            border: 1px solid #d0ffe8;
          }
          
          &:active {
            transform: scale(0.95);
          }
        }
        
        .action-icon {
          width: 50rpx;
          height: 50rpx;
        }
        
        .action-text {
          font-size: 30rpx;
          color: #333;
          font-weight: 500;
        }
      }
    }
  }
}

// 遮罩层
.mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 999;
  animation: fadeIn 0.3s ease;
}

@keyframes slideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
