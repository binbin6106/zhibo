<template>
  <view class="page">
    <view class="header">
      <view class="header-left">
        <text class="badge">LIVE</text>
        <view class="title-wrap">
          <text class="title">智能装备赛道</text>
          <text class="subtitle">智能制造 · 即时下单</text>
        </view>
      </view>
      <view class="settings-entry" @click="openSettings">
        <text class="settings-icon">⚙️</text>
      </view>
    </view>

    <view class="player-card">
      <live-player
        id="rtmpPlayer"
        class="player"
        :src="streamUrl"
        mode="live"
        autoplay
        object-fit="contain"
        @statechange="handlePlayerStateChange"
        @error="handlePlayerError"
      />
      <view class="live-info">
        <text class="live-title">基于电商订单驱动的柔性智能制造系统</text>
        <text class="live-desc">第十八届山东省职业院校技能大赛</text>
      </view>
    </view>

    <view class="section">
      <view class="section-title">
        <text class="dot" />
        <text>商品精选</text> 
      </view>
      <scroll-view class="product-list" scroll-x>
        <view
          v-for="product in products"
          :key="product.id"
          class="product-card"
          :class="{ active: selectedProduct && selectedProduct.id === product.id }"
          @click="selectProduct(product)"
        >
          <image class="product-image" :src="product.image" mode="aspectFill" />
          <view class="product-info">
            <text class="product-name">{{ product.name }}</text>
            <text class="product-tag">{{ product.tag }}</text>
            <view class="product-footer">
              <text class="product-price">¥{{ product.price }}</text>
              <text class="choose-text">{{ selectedProduct && selectedProduct.id === product.id ? '已选' : '选择' }}</text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>

    <view class="action-card">
      <view>
        <text class="action-title">下单中心</text>
        <text class="action-desc">
          请选择心仪的商品并确认下单，订单状态会实时显示
        </text>
      </view>
      <button class="order-btn" type="primary" @click="placeOrder">
        {{ selectedProduct ? `下单：${selectedProduct.name}` : '请选择商品后下单' }}
      </button>
    </view>

    <view class="status-card" v-if="orderStatus">
      <view class="status-title">订单状态</view>
      <text class="status-text">{{ orderStatus }}</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      orderStatus: '',
      socketOpen: false,
      socketTask: null,
      streamUrl: 'rtmp://localhost/live/stream',
      websocketUrl: 'ws://localhost:8080',
      selectedProduct: null,
      products: [
        {
          id: 'P-1001',
          name: '光伏板',
          price: 199,
          tag: 'guangfu',
          image: 'https://q4.itc.cn/images01/20241223/084b3f28c95647c896a31f81f15e5901.png',
        }
      ],
    };
  },
  onLoad() {
    this.loadSettings();
  },
  onShow() {
    this.loadSettings();
    this.connectIfReady();
  },
  onUnload() {
    this.closeSocket();
  },
  methods: {
    loadSettings() {
      const storedStream = uni.getStorageSync('streamUrl');
      const storedWs = uni.getStorageSync('websocketUrl');
      this.streamUrl = storedStream || 'rtmp://localhost/live/stream';
      this.websocketUrl = storedWs || 'ws://localhost:8080';
    },
    connectIfReady() {
      if (!this.streamUrl || !this.websocketUrl) {
        this.orderStatus = '请前往设置填写直播地址和 WebSocket';
        return;
      }

      this.orderStatus = '正在连接直播服务...';
      this.connectWebSocket();
    },
    connectWebSocket() {
      this.closeSocket();

      const url = this.websocketUrl;
      const socketTask = uni.connectSocket({
        url,
        success: () => {
          console.log('WebSocket连接发起成功');
        },
      });

      this.socketTask = socketTask;

      socketTask.onOpen(() => {
        this.socketOpen = true;
        this.orderStatus = '已连接直播间，等待下单';
        console.log('WebSocket已打开');
      });

      socketTask.onMessage((message) => {
        this.orderStatus = message.data;
      });

      socketTask.onClose(() => {
        this.socketOpen = false;
        console.log('WebSocket已关闭');
      });

      socketTask.onError((err) => {
        this.socketOpen = false;
        this.orderStatus = '连接出错，请检查地址后重试';
        console.error('WebSocket连接出错', err);
      });
    },
    openSettings() {
      uni.navigateTo({
        url: '/pages/settings/index',
      });
    },
    closeSocket() {
      if (this.socketTask) {
        this.socketTask.close({
          code: 1000,
          reason: 'reconnect',
        });
        this.socketTask = null;
      }
    },
    selectProduct(product) {
      this.selectedProduct = product;
    },
    placeOrder() {
      if (!this.selectedProduct) {
        uni.showToast({
          title: '请先选择商品',
          icon: 'none',
        });
        return;
      }

      const orderInfo = `商品ID: ${this.selectedProduct.id}`;
      if (this.socketOpen && this.socketTask) {
        this.socketTask.send({
          data: orderInfo,
          success: () => {
            this.orderStatus = `${this.selectedProduct.name} 下单成功，等待确认...`;
          },
          fail: () => {
            this.orderStatus = '下单失败，请稍后重试';
          },
        });
      } else {
        this.orderStatus = '连接未建立，无法下单';
      }
    },
    handlePlayerStateChange(e) {
      console.log('RTMP 播放状态变更：', e);
    },
    handlePlayerError(err) {
      console.error('RTMP 播放出错：', err);
      this.orderStatus = '直播播放出现问题，请检查 RTMP 地址';
    },
  },
};
</script>

<style scoped>
.page {
  min-height: 100vh;
  padding: 20px 16px 40px;
  background: linear-gradient(180deg, #0f172a 0%, #111827 40%, #0b1220 100%);
  color: #fff;
  box-sizing: border-box;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
}

.header-left {
  display: flex;
  align-items: center;
}

.badge {
  background: #ef4444;
  color: #fff;
  font-weight: 700;
  padding: 6px 10px;
  border-radius: 10px;
  font-size: 12px;
  letter-spacing: 1px;
  margin-right: 10px;
}

.title-wrap {
  display: flex;
  flex-direction: column;
}

.title {
  font-size: 22px;
  font-weight: 700;
  color: #f9fafb;
}

.subtitle {
  font-size: 13px;
  color: #cbd5e1;
  margin-top: 4px;
}

.settings-entry {
  width: 36px;
  height: 36px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #e5e7eb;
}

.settings-icon {
  font-size: 18px;
}

.player-card {
  background: #0b1220;
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.45);
  margin-bottom: 16px;
}

.player {
  width: 100%;
  height: 220px;
  background: #000;
}

.live-info {
  padding: 14px 16px 18px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.live-title {
  font-size: 16px;
  font-weight: 700;
}

.live-desc {
  font-size: 13px;
  color: #cbd5e1;
}

.section {
  margin-bottom: 18px;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #e5e7eb;
  font-size: 15px;
  margin-bottom: 12px;
}

.dot {
  width: 8px;
  height: 8px;
  background: #22c55e;
  border-radius: 999px;
}

.product-list {
  white-space: nowrap;
  display: flex;
}

.product-card {
  display: inline-flex;
  flex-direction: column;
  width: 180px;
  margin-right: 12px;
  border-radius: 14px;
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.08) 0%, rgba(255, 255, 255, 0.03) 100%);
  border: 1px solid rgba(255, 255, 255, 0.08);
  overflow: hidden;
  transition: all 0.2s ease;
}

.product-card.active {
  border-color: #22c55e;
  box-shadow: 0 8px 24px rgba(34, 197, 94, 0.25);
  transform: translateY(-2px);
}

.product-image {
  width: 100%;
  height: 120px;
  background: #0f172a;
}

.product-info {
  padding: 10px 12px 12px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.product-name {
  font-size: 15px;
  font-weight: 700;
  color: #f8fafc;
  white-space: normal;
}

.product-tag {
  font-size: 12px;
  color: #a5f3fc;
  background: rgba(34, 211, 238, 0.14);
  padding: 4px 6px;
  border-radius: 8px;
  align-self: flex-start;
}

.product-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.product-price {
  font-size: 16px;
  font-weight: 700;
  color: #f97316;
}

.choose-text {
  font-size: 13px;
  color: #cbd5e1;
}

.action-card {
  background: linear-gradient(135deg, rgba(34, 197, 94, 0.14), rgba(59, 130, 246, 0.12));
  border-radius: 16px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.action-title {
  font-size: 16px;
  font-weight: 700;
}

.action-desc {
  font-size: 13px;
  color: #d1d5db;
  margin-top: 6px;
}

.order-btn {
  margin-top: 4px;
  background: linear-gradient(90deg, #22c55e, #10b981);
  color: #0b1220;
  font-weight: 700;
  border: none;
  border-radius: 12px;
}

.status-card {
  margin-top: 16px;
  padding: 14px 16px;
  background: rgba(255, 255, 255, 0.04);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.status-title {
  font-size: 14px;
  color: #a5b4fc;
  margin-bottom: 8px;
}

.status-text {
  font-size: 15px;
  color: #f1f5f9;
}
</style>
