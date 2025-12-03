<template>
  <view class="page">
    <view class="header">
      <text class="badge">LIVE</text>
      <view class="title-wrap">
        <text class="title">精选直播间</text>
        <text class="subtitle">实时讲解 · 即时下单</text>
      </view>
    </view>

    <view class="player-card">
      <video
        id="rtmpVideo"
        class="player"
        :src="'http://localhost:8085/hls/test.m3u8'"
        controls
        autoplay
        :enable-progress-gesture="false"
        object-fit="contain"
      ></video>
      <view class="live-info">
        <text class="live-title">今日主推好物，限时优惠</text>
        <text class="live-desc">跟随主播讲解，点击下方商品即可快速下单</text>
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
      selectedProduct: null,
      products: [
        {
          id: 'P-1001',
          name: '无线蓝牙耳机',
          price: 199,
          tag: '人气爆款',
          image: 'https://img.alicdn.com/imgextra/i1/2206689600457/O1CN01EG38FV1Moq2xG3pVq_!!2206689600457-0-lubanu-s.jpg',
        },
        {
          id: 'P-1002',
          name: '智能手表',
          price: 329,
          tag: '限时直降',
          image: 'https://img.alicdn.com/imgextra/i3/2207021320110/O1CN01ev4Z4P1x7lHrT7cwQ_!!2207021320110-0-lubanu-s.jpg',
        },
        {
          id: 'P-1003',
          name: '迷你榨汁杯',
          price: 149,
          tag: '直播专享',
          image: 'https://img.alicdn.com/imgextra/i1/3526327762/O1CN01ms05591Id0a8ggxqv_!!0-item_pic.jpg',
        },
        {
          id: 'P-1004',
          name: '高颜值保温杯',
          price: 89,
          tag: '礼盒装',
          image: 'https://img.alicdn.com/imgextra/i3/2208688884926/O1CN01x6DShv1uJEg5udweT_!!0-item_pic.jpg',
        },
      ],
    };
  },
  onLoad() {
    this.connectWebSocket();
  },
  methods: {
    connectWebSocket() {
      const socketTask = uni.connectSocket({
        url: 'ws://localhost:8080',
        success: () => {
          console.log('WebSocket连接发起成功');
        },
      });

      socketTask.onOpen(() => {
        this.socketOpen = true;
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
        console.error('WebSocket连接出错', err);
      });
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
      if (this.socketOpen) {
        uni.sendSocketMessage({
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
  margin-bottom: 18px;
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
