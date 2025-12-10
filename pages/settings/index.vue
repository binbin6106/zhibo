<template>
  <view class="page">
    <view class="header">
      <text class="badge">SETTINGS</text>
      <view class="title-wrap">
        <text class="title">直播配置</text>
        <text class="subtitle">修改直播播放地址与 WebSocket 连接</text>
      </view>
    </view>

    <view class="settings-card">
      <view class="form-field">
        <text class="field-label">直播地址</text>
        <input
          v-model.trim="streamUrl"
          class="field-input"
          placeholder="请输入直播流地址，如 rtmp://..."
          placeholder-class="field-placeholder"
        />
      </view>

      <view class="form-field">
        <text class="field-label">WebSocket 地址</text>
        <input
          v-model.trim="websocketUrl"
          class="field-input"
          placeholder="请输入 WebSocket 地址，如 ws://..."
          placeholder-class="field-placeholder"
        />
      </view>

      <button class="apply-btn" type="primary" @click="saveSettings">
        保存并返回
      </button>
    </view>
  </view>
</template>

<script>
const STREAM_KEY = 'streamUrl';
const WS_KEY = 'websocketUrl';

export default {
  data() {
    return {
      streamUrl: '',
      websocketUrl: '',
    };
  },
  onLoad() {
    this.loadSettings();
  },
  methods: {
    loadSettings() {
      const storedStream = uni.getStorageSync(STREAM_KEY);
      const storedWs = uni.getStorageSync(WS_KEY);
      this.streamUrl = storedStream || 'rtmp://localhost/live/stream';
      this.websocketUrl = storedWs || 'ws://localhost:8080';
    },
    saveSettings() {
      if (!this.streamUrl) {
        uni.showToast({
          title: '请填写直播地址',
          icon: 'none',
        });
        return;
      }

      if (!this.websocketUrl) {
        uni.showToast({
          title: '请填写 WebSocket 地址',
          icon: 'none',
        });
        return;
      }

      uni.setStorageSync(STREAM_KEY, this.streamUrl);
      uni.setStorageSync(WS_KEY, this.websocketUrl);

      uni.showToast({
        title: '保存成功',
        icon: 'success',
      });

      setTimeout(() => {
        uni.navigateBack();
      }, 300);
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
  background: #22c55e;
  color: #0b1220;
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

.settings-card {
  padding: 16px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.form-field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.field-label {
  font-size: 13px;
  color: #e5e7eb;
}

.field-input {
  width: 100%;
  padding: 10px 12px;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 10px;
  color: #fff;
  font-size: 14px;
}

.field-placeholder {
  color: #9ca3af;
}

.apply-btn {
  margin-top: 4px;
  background: linear-gradient(90deg, #22c55e, #10b981);
  color: #0b1220;
  font-weight: 700;
  border: none;
  border-radius: 12px;
}
</style>
