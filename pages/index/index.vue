<template>
  <view class="container">
    <text class="title">直播平台</text>
    
    <!-- RTMP视频流播放 -->
    <video 
      id="rtmpVideo" 
      :src="'http://localhost:8085/hls/test.m3u8'" 
      controls 
      autoplay 
      width="100%" 
      height="300px">
    </video>

    <!-- 下单按钮 -->
    <button @click="placeOrder">下单</button>

    <!-- 订单状态 -->
    <view>{{ orderStatus }}</view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      orderStatus: '',  // 用于显示订单状态
      socketOpen: false,
    };
  },
  onLoad() {
    this.connectWebSocket();
  },
  methods: {
    // WebSocket连接
    connectWebSocket() {
      uni.connectSocket({
        url: 'ws://localhost:8080',  // 连接到服务端WebSocket
        success: () => {
          console.log('WebSocket连接成功！');
        },
      });

      // 接收服务端消息
      uni.onSocketMessage((message) => {
        this.orderStatus = message.data;
      });
    },
    // 下单按钮点击事件
    placeOrder() {
      const orderInfo = "商品ID: 12345";  // 模拟商品ID
      if (this.socketOpen) {
        uni.sendSocketMessage({
          data: orderInfo,
        });
      }
    },
  },
};
</script>

<style scoped>
.container {
  padding: 20px;
  text-align: center;
}

.title {
  font-size: 24px;
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
}

#orderStatus {
  margin-top: 20px;
  font-size: 18px;
}
</style>
