---
title: oss上传功能文档及实现
tags:
  - oss上传功能文档及实现
author: Eric
date: 2026-2-26
comments: false
cover: /img/v4.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

- [一、OSS上传基础概念](#一oss上传基础概念)
  - [1.1 什么是OSS？](#11-什么是oss)
  - [1.2 为什么需要前端直传？](#12-为什么需要前端直传)
- [二、上传模式对比](#二上传模式对比)
- [三、准备工作](#三准备工作)
  - [3.1 OSS Bucket配置](#31-oss-bucket配置)
  - [3.2 RAM角色配置](#32-ram角色配置)
- [四、实战：STS临时凭证模式](#四实战sts临时凭证模式)
  - [4.1 后端STS服务（Node.js实现）](#41-后端sts服务nodejs实现)
  - [4.2 前端OSS工具类](#42-前端oss工具类)
- [五、框架集成示例](#五框架集成示例)
  - [5.1 Vue 3组件示例](#51-vue-3组件示例)
- [六、高级功能实现](#六高级功能实现)
  - [6.1 图片压缩上传](#61-图片压缩上传)
  - [6.2 分片上传（大文件）](#62-分片上传大文件)
  - [6.3 断点续传](#63-断点续传)
- [七、最佳实践指南](#七最佳实践指南)
  - [7.1 错误处理](#71-错误处理)
  - [7.2 性能优化](#72-性能优化)
- [八、常见问题排查](#八常见问题排查)
  - [8.1 错误码对照表](#81-错误码对照表)
- [九、总结](#九总结)
  - [9.1 核心要点总结](#91-核心要点总结)
    - [安全性优先](#安全性优先)
    - [用户体验](#用户体验)
    - [性能优化](#性能优化)
    - [可维护性](#可维护性)

## 一、OSS上传基础概念

### 1.1 什么是OSS？

OSS（Object Storage Service）是阿里云提供的对象存储服务，可以存储和管理任意类型的文件，如图片、视频、文档等。它具有高可用、高可靠、低成本的特点，广泛应用于静态资源托管、文件备份、大数据分析等场景。

### 1.2 为什么需要前端直传？

**传统上传方式：**
缺点：文件经过应用服务器中转，增加服务器压力，带宽成为瓶颈，上传速度慢

**前端直传方式：**
优点：

- 减轻服务器压力
- 上传速度快（利用OSS全球加速）
- 节省服务器带宽成本
- 用户体验更好

## 二、上传模式对比

| 模式 | 安全性 | 实现复杂度 | 适用场景 | 优点 | 缺点 |

|------|--------|------------|----------|------|------|

| **STS临时凭证** | ⭐⭐⭐⭐⭐ | 中等 | **生产环境推荐** | 安全可控、支持权限控制、凭证自动过期 | 需要后端配合 |
| **签名URL** | ⭐⭐⭐⭐ | 简单 | 临时分享、公开文件 | 无需后端存储凭证、适合临时访问 | 有时效性限制 |
| **直接AK** | ⭐ | 最简单 | **仅限本地测试** | 快速验证功能 | 密钥暴露风险极高 |

## 三、准备工作

### 3.1 OSS Bucket配置

在阿里云OSS控制台进行以下配置：

### 3.2 RAM角色配置

```yaml
1. 创建Bucket:
   - 名称: your-bucket-name
   - 地域: oss-cn-hangzhou
   - 读写权限: 私有（配合STS使用）

2. 配置CORS规则:
   - 来源: * 或 你的域名
   - 方法: GET, PUT, POST, DELETE, HEAD
   - 头: *
   - 暴露头: ETag
   1. 创建RAM用户:
   - 授权策略: AliyunSTSAssumeRoleAccess

2. 创建RAM角色:
   - 信任策略: RAM用户
   - 授权策略: 自定义OSS上传权限

3. 自定义策略示例:
   {
     "Version": "1",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "oss:PutObject",
           "oss:GetObject"
         ],
         "Resource": [
           "acs:oss:*:*:your-bucket/*"
         ]
       }
     ]
   }
```

## 四、实战：STS临时凭证模式

### 4.1 后端STS服务（Node.js实现）

```js
// server/sts-server.js
const express = require('express');
const OSS = require('ali-oss');
const cors = require('cors');
const app = express();

// 启用CORS
app.use(cors());

// RAM角色配置
const config = {
  region: 'oss-cn-hangzhou',
  accessKeyId: process.env.ACCESS_KEY_ID, // 从环境变量读取
  accessKeySecret: process.env.ACCESS_KEY_SECRET,
  roleArn: 'acs:ram::1234567890:role/oss-upload-role',
  bucket: 'your-bucket',
  durationSeconds: 3600, // 1小时有效期
};

/**
 * 获取STS临时凭证接口
 * @route GET /api/sts/token
 */
app.get('/api/sts/token', async (req, res) => {
  try {
    const { STS } = require('ali-oss');
    const sts = new STS({
      accessKeyId: config.accessKeyId,
      accessKeySecret: config.accessKeySecret,
    });

    // 获取STS临时凭证
    const { credentials } = await sts.assumeRole(
      config.roleArn,
      '', // 可以传递临时策略
      config.durationSeconds,
      'upload-session-' + Date.now(),
    );

    // 返回凭证
    res.json({
      code: 0,
      data: {
        AccessKeyId: credentials.AccessKeyId,
        AccessKeySecret: credentials.AccessKeySecret,
        SecurityToken: credentials.SecurityToken,
        Expiration: credentials.Expiration,
      },
      message: 'success',
    });
  } catch (error) {
    console.error('获取STS凭证失败:', error);
    res.status(500).json({
      code: 500,
      message: error.message,
    });
  }
});

/**
 * 健康检查接口
 */
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', timestamp: Date.now() });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`STS服务启动成功：http://localhost:${PORT}`);
});
```

### 4.2 前端OSS工具类

```js
// src/utils/ossUploader.js
import OSS from 'ali-oss';

/**
 * OSS上传核心类
 * 封装了STS认证、文件上传、进度回调等功能
 */
class OSSUploader {
  constructor(options = {}) {
    this.client = null;
    this.config = {
      region: options.region || 'oss-cn-hangzhou',
      bucket: options.bucket || 'your-bucket-name',
      secure: true, // 启用HTTPS
      basePath: this._generateBasePath(),
      stsUrl: options.stsUrl || 'http://localhost:3000/api/sts/token',
      ...options,
    };
  }

  /**
   * 生成基础路径（按年月组织文件）
   * @private
   */
  _generateBasePath() {
    const date = new Date();
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `uploads/${year}/${month}/${day}/`;
  }

  /**
   * 获取STS临时凭证
   * @returns {Promise<Object>} STS凭证
   * @private
   */
  async _fetchSTSToken() {
    try {
      const response = await fetch(this.config.stsUrl, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
        },
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();

      if (result.code !== 0) {
        throw new Error(result.message || '获取STS凭证失败');
      }

      return result.data;
    } catch (error) {
      console.error('获取STS凭证失败:', error);
      throw error;
    }
  }

  /**
   * 初始化OSS客户端
   * @returns {Promise<boolean>} 初始化结果
   */
  async initClient() {
    try {
      const stsToken = await this._fetchSTSToken();

      this.client = new OSS({
        region: this.config.region,
        bucket: this.config.bucket,
        accessKeyId: stsToken.AccessKeyId,
        accessKeySecret: stsToken.AccessKeySecret,
        stsToken: stsToken.SecurityToken,
        secure: this.config.secure,
        timeout: 120000, // 2分钟超时

        // 自动刷新Token
        refreshSTSToken: async () => {
          const newToken = await this._fetchSTSToken();
          return {
            accessKeyId: newToken.AccessKeyId,
            accessKeySecret: newToken.AccessKeySecret,
            stsToken: newToken.SecurityToken,
          };
        },
        refreshSTSTokenInterval: 300000, // 5分钟刷新一次
      });

      return true;
    } catch (error) {
      console.error('初始化OSS客户端失败:', error);
      return false;
    }
  }

  /**
   * 生成唯一的文件名
   * @param {File} file - 文件对象
   * @returns {string} 唯一文件名
   */
  generateFileName(file) {
    const timestamp = Date.now();
    const random = Math.random().toString(36).substring(2, 10);
    const ext = file.name.substring(file.name.lastIndexOf('.'));
    return `${this.config.basePath}${timestamp}-${random}${ext}`;
  }

  /**
   * 验证文件
   * @param {File} file - 文件对象
   * @param {Object} rules - 验证规则
   * @returns {Object} 验证结果
   */
  validateFile(file, rules = {}) {
    const {
      maxSize = 10 * 1024 * 1024, // 默认10MB
      allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'video/mp4'],
    } = rules;

    if (file.size > maxSize) {
      return {
        valid: false,
        message: `文件大小不能超过${maxSize / 1024 / 1024}MB`,
      };
    }

    if (allowedTypes.length > 0 && !allowedTypes.includes(file.type)) {
      return {
        valid: false,
        message: `不支持的文件类型：${file.type}`,
      };
    }

    return { valid: true };
  }

  /**
   * 上传单个文件
   * @param {File} file - 文件对象
   * @param {Object} callbacks - 回调函数
   */
  async uploadFile(file, callbacks = {}) {
    const {
      onProgress = () => {},
      onSuccess = () => {},
      onError = () => {},
      onStart = () => {},
    } = callbacks;

    try {
      onStart(file);

      // 确保客户端已初始化
      if (!this.client) {
        const initSuccess = await this.initClient();
        if (!initSuccess) {
          throw new Error('客户端初始化失败');
        }
      }

      // 生成文件名
      const fileName = this.generateFileName(file);

      // 执行上传
      const result = await this.client.put(fileName, file, {
        progress: (percentage) => {
          const progress = Math.floor(percentage * 100);
          onProgress(progress);
        },
        headers: {
          'x-oss-object-acl': 'public-read', // 设置公共读权限
          'x-oss-meta-originalname': encodeURIComponent(file.name), // 保存原始文件名
        },
        meta: {
          size: file.size,
          type: file.type,
          uploadTime: new Date().toISOString(),
        },
      });

      // 构造返回信息
      const fileInfo = {
        url: result.url,
        name: result.name,
        originalName: file.name,
        size: file.size,
        type: file.type,
        etag: result.res.headers.etag,
        uploadTime: Date.now(),
      };

      onSuccess(fileInfo);
      return fileInfo;
    } catch (error) {
      console.error('上传失败:', error);
      onError(error);
      throw error;
    }
  }

  /**
   * 批量上传文件
   * @param {File[]} files - 文件数组
   * @param {Object} callbacks - 回调函数
   */
  async uploadFiles(files, callbacks = {}) {
    const {
      onBatchProgress = () => {},
      onFileComplete = () => {},
      onError = () => {},
    } = callbacks;

    const results = [];
    const total = files.length;

    for (let i = 0; i < files.length; i++) {
      const file = files[i];

      try {
        const result = await this.uploadFile(file, {
          onProgress: (progress) => {
            onBatchProgress(((i + progress / 100) / total) * 100);
          },
          onSuccess: (fileInfo) => {
            results.push({ success: true, file: fileInfo, index: i });
            onFileComplete({ success: true, file: fileInfo, index: i });
          },
        });
      } catch (error) {
        results.push({ success: false, error: error.message, index: i });
        onFileComplete({ success: false, error: error.message, index: i });
        onError(error);
      }

      onBatchProgress(((i + 1) / total) * 100);
    }

    return results;
  }

  /**
   * 取消上传
   */
  cancelUpload() {
    if (this.client) {
      this.client.cancel();
    }
  }
}
export default OSSUploader;
```

## 五、框架集成示例

### 5.1 Vue 3组件示例

```js
<!-- src/components/OSSUploader.vue -->
<template>
  <div class="oss-uploader">
    <!-- 上传区域 -->
    <div
      class="upload-area"
      :class="{
        'drag-active': dragActive,
        uploading: uploading
      }"
      @click="triggerFileInput"
      @dragenter="handleDragEnter"
      @dragleave="handleDragLeave"
      @dragover.prevent
      @drop.prevent="handleDrop"
    >
      <input
        ref="fileInput"
        type="file"
        :multiple="multiple"
        :accept="accept"
        @change="handleFileChange"
        style="display: none"
      />

      <div class="upload-content">
        <svg class="upload-icon" viewBox="0 0 24 24">
          <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
        </svg>

        <template v-if="uploading">
          <div class="upload-progress">
            <div class="progress-bar">
              <div
                class="progress-fill"
                :style="{ width: progress + '%' }"
              />
            </div>
            <span class="progress-text">{{ progress.toFixed(1) }}%</span>
          </div>
        </template>

        <template v-else>
          <p class="upload-text">
            <strong>点击或拖拽文件</strong> 到此区域上传
          </p>
          <p class="upload-hint">
            支持格式：{{ accept }}，单个文件最大 {{ maxSizeMB }}MB
          </p>
        </template>
      </div>
    </div>

    <!-- 文件列表 -->
    <div v-if="uploadedFiles.length" class="file-list">
      <h4>已上传文件 ({{ uploadedFiles.length }})</h4>
      <div class="file-grid">
        <div v-for="(file, index) in uploadedFiles" :key="index" class="file-item">
          <img
            v-if="isImage(file.type)"
            :src="file.url"
            :alt="file.originalName"
            class="file-preview"
            loading="lazy"
          />
          <div v-else class="file-icon">
            📄
          </div>
          <div class="file-info">
            <a
              :href="file.url"
              target="_blank"
              rel="noopener noreferrer"
              class="file-name"
              :title="file.originalName"
            >
              {{ file.originalName }}
            </a>
            <span class="file-size">{{ formatSize(file.size) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import OSSUploader from '../utils/ossUploader';

export default {
  name: 'OSSUploader',
  props: {
    multiple: {
      type: Boolean,
      default: false
    },
    accept: {
      type: String,
      default: 'image/*,video/*'
    },
    maxSize: {
      type: Number,
      default: 10 * 1024 * 1024 // 10MB
    },
    stsUrl: {
      type: String,
      default: 'http://localhost:3000/api/sts/token'
    }
  },
  emits: ['success', 'error', 'progress'],

  setup(props, { emit }) {
    const fileInput = ref(null);
    const uploading = ref(false);
    const progress = ref(0);
    const uploadedFiles = ref([]);
    const dragActive = ref(false);
    const uploader = ref(null);

    // 计算最大文件大小（MB）
    const maxSizeMB = computed(() => (props.maxSize / 1024 / 1024).toFixed(1));

    /**
     * 初始化上传器
     */
    const initUploader = async () => {
      if (!uploader.value) {
        uploader.value = new OSSUploader({
          stsUrl: props.stsUrl
        });
        await uploader.value.initClient();
      }
    };

    /**
     * 触发文件选择
     */
    const triggerFileInput = () => {
      if (!uploading.value) {
        fileInput.value.click();
      }
    };

    /**
     * 处理拖拽进入
     */
    const handleDragEnter = (e) => {
      e.preventDefault();
      dragActive.value = true;
    };

    /**
     * 处理拖拽离开
     */
    const handleDragLeave = (e) => {
      e.preventDefault();
      dragActive.value = false;
    };

    /**
     * 处理拖拽释放
     */
    const handleDrop = (e) => {
      e.preventDefault();
      dragActive.value = false;

      if (!uploading.value && e.dataTransfer.files.length > 0) {
        handleFiles(e.dataTransfer.files);
      }
    };

    /**
     * 处理文件选择变化
     */
    const handleFileChange = (e) => {
      handleFiles(e.target.files);
    };

    /**
     * 处理文件
     */
    const handleFiles = async (fileList) => {
      const files = Array.from(fileList);

      // 文件大小验证
      const validFiles = files.filter(file => {
        if (file.size > props.maxSize) {
          alert(`文件 ${file.name} 超过大小限制 (${maxSizeMB.value}MB)`);
          return false;
        }
        return true;
      });

      if (validFiles.length === 0) return;

      uploading.value = true;
      progress.value = 0;

      try {
        await initUploader();

        for (let i = 0; i < validFiles.length; i++) {
          const file = validFiles[i];

          await uploader.value.uploadFile(file, {
            onProgress: (p) => {
              progress.value = p;
              emit('progress', p);
            },
            onSuccess: (fileInfo) => {
              uploadedFiles.value.push(fileInfo);
              emit('success', fileInfo);
            },
            onError: (error) => {
              console.error('文件上传失败:', error);
              emit('error', { file, error });
            }
          });
        }
      } catch (error) {
        console.error('上传过程异常:', error);
        emit('error', error);
      } finally {
        uploading.value = false;
        progress.value = 0;
        dragActive.value = false;
        // 清空input，允许重新选择同一文件
        if (fileInput.value) {
          fileInput.value.value = '';
        }
      }
    };

    /**
     * 判断是否为图片
     */
    const isImage = (fileType) => {
      return fileType?.startsWith('image/');
    };

    /**
     * 格式化文件大小
     */
    const formatSize = (bytes) => {
      if (bytes === 0) return '0 B';
      const k = 1024;
      const sizes = ['B', 'KB', 'MB', 'GB'];
      const i = Math.floor(Math.log(bytes) / Math.log(k));
      return (bytes / Math.pow(k, i)).toFixed(2) + ' ' + sizes[i];
    };

    return {
      fileInput,
      uploading,
      progress,
      uploadedFiles,
      dragActive,
      maxSizeMB,
      triggerFileInput,
      handleDragEnter,
      handleDragLeave,
      handleDrop,
      handleFileChange,
      isImage,
      formatSize
    };
  }
};
</script>

<style scoped>
/* src/components/OSSUploader.css */
.oss-uploader {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.upload-area {
  border: 2px dashed #d9d9d9;
  border-radius: 8px;
  padding: 40px 20px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  background-color: #fafafa;
}

.upload-area:hover {
  border-color: #1890ff;
  background-color: #f5f5f5;
}

.upload-area.drag-active {
  border-color: #1890ff;
  background-color: #e6f7ff;
}

.upload-area.uploading {
  cursor: not-allowed;
  opacity: 0.7;
}

.upload-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.upload-icon {
  width: 48px;
  height: 48px;
  fill: #999;
  transition: fill 0.3s;
}

.upload-area:hover .upload-icon {
  fill: #1890ff;
}

.upload-text {
  margin: 0;
  color: #333;
  font-size: 16px;
}

.upload-text strong {
  color: #1890ff;
}

.upload-hint {
  margin: 0;
  color: #999;
  font-size: 14px;
}

/* 进度条 */
.upload-progress {
  width: 100%;
  max-width: 300px;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background-color: #f0f0f0;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 8px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #1890ff, #40a9ff);
  transition: width 0.3s ease;
  border-radius: 4px;
}

.progress-text {
  color: #1890ff;
  font-size: 14px;
  font-weight: 500;
}

/* 文件列表 */
.file-list {
  margin-top: 24px;
}

.file-list h4 {
  margin: 0 0 12px 0;
  color: #333;
  font-size: 16px;
  font-weight: 500;
}

.file-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
}

.file-item {
  border: 1px solid #f0f0f0;
  border-radius: 4px;
  overflow: hidden;
  transition: box-shadow 0.3s;
  background: white;
}

.file-item:hover {
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.file-preview {
  width: 100%;
  height: 150px;
  object-fit: cover;
  background-color: #f5f5f5;
}

.file-icon {
  width: 100%;
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f5f5f5;
  font-size: 48px;
}

.file-info {
  padding: 8px;
  border-top: 1px solid #f0f0f0;
}

.file-name {
  display: block;
  color: #333;
  text-decoration: none;
  font-size: 13px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-bottom: 4px;
}

.file-name:hover {
  color: #1890ff;
}

.file-size {
  color: #999;
  font-size: 11px;
}

/* 加载动画 */
@keyframes pulse {
  0% { opacity: 1; }
  50% { opacity: 0.5; }
  100% { opacity: 1; }
}

.uploading .upload-icon {
  animation: pulse 1.5s ease-in-out infinite;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .upload-area {
    padding: 30px 15px;
  }

  .file-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 12px;
  }

  .file-preview,
  .file-icon {
    height: 120px;
  }
}

@media (max-width: 480px) {
  .file-grid {
    grid-template-columns: 1fr;
  }
}
</style>
```

## 六、高级功能实现

### 6.1 图片压缩上传

```js
// src/utils/imageCompressor.js
/**
 * 图片压缩工具类
 */
class ImageCompressor {
  /**
   * 压缩图片
   * @param {File} file - 原始图片文件
   * @param {Object} options - 压缩选项
   */
  static async compress(file, options = {}) {
    const {
      maxWidth = 1920,
      maxHeight = 1080,
      quality = 0.8,
      format = 'image/jpeg',
    } = options;

    // 如果文件不是图片，直接返回
    if (!file.type.startsWith('image/')) {
      return file;
    }

    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);

      reader.onload = (e) => {
        const img = new Image();
        img.src = e.target.result;

        img.onload = () => {
          // 计算压缩后的尺寸（保持宽高比）
          let { width, height } = img;

          if (width > maxWidth) {
            height = Math.round((maxWidth / width) * height);
            width = maxWidth;
          }

          if (height > maxHeight) {
            width = Math.round((maxHeight / height) * width);
            height = maxHeight;
          }

          // 创建canvas进行压缩
          const canvas = document.createElement('canvas');
          canvas.width = width;
          canvas.height = height;

          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, width, height);

          // 转换为blob
          canvas.toBlob(
            (blob) => {
              // 创建新的File对象
              const compressedFile = new File([blob], file.name, {
                type: format,
                lastModified: Date.now(),
              });

              // 计算压缩率
              const compressionRate = (
                ((file.size - blob.size) / file.size) *
                100
              ).toFixed(1);
              console.log(`压缩完成：${compressionRate}% 节省空间`);

              resolve(compressedFile);
            },
            format,
            quality,
          );
        };

        img.onerror = (error) => {
          reject(new Error('图片加载失败：' + error.message));
        };
      };

      reader.onerror = (error) => {
        reject(new Error('文件读取失败：' + error.message));
      };
    });
  }

  /**
   * 获取图片信息
   */
  static async getImageInfo(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);

      reader.onload = (e) => {
        const img = new Image();
        img.src = e.target.result;

        img.onload = () => {
          resolve({
            width: img.width,
            height: img.height,
            type: file.type,
            size: file.size,
            name: file.name,
          });
        };

        img.onerror = reject;
      };

      reader.onerror = reject;
    });
  }
}

export default ImageCompressor;
```

### 6.2 分片上传（大文件）

```js
// src/utils/chunkUploader.js
/**
 * 分片上传器（用于大文件）
 */
class ChunkUploader {
  constructor(uploader, options = {}) {
    this.uploader = uploader;
    this.chunkSize = options.chunkSize || 1024 * 1024; // 默认1MB
    this.concurrent = options.concurrent || 3; // 默认3个并发
    this.retryCount = options.retryCount || 3; // 失败重试次数
  }

  /**
   * 分片上传
   * @param {File} file - 文件对象
   * @param {string} fileName - 文件名
   * @param {Object} callbacks - 回调函数
   */
  async upload(file, fileName, callbacks = {}) {
    const {
      onProgress = () => {},
      onChunkComplete = () => {},
      onComplete = () => {},
    } = callbacks;

    // 计算分片数量
    const chunkCount = Math.ceil(file.size / this.chunkSize);
    console.log(`文件大小：${file.size}，分片数量：${chunkCount}`);

    try {
      // 初始化分片上传
      const { uploadId } =
        await this.uploader.client.initMultipartUpload(fileName);

      // 准备上传队列
      const chunks = [];
      for (let i = 0; i < chunkCount; i++) {
        const start = i * this.chunkSize;
        const end = Math.min(start + this.chunkSize, file.size);
        chunks.push({
          number: i + 1,
          data: file.slice(start, end),
          retries: 0,
        });
      }

      // 上传结果
      const uploadedParts = [];
      const failedChunks = [];

      // 并发控制
      const queue = [...chunks];
      const running = [];

      const uploadChunk = async (chunk) => {
        try {
          console.log(`上传分片 ${chunk.number}/${chunkCount}`);

          const result = await this.uploader.client.uploadPart(
            fileName,
            uploadId,
            chunk.number,
            chunk.data,
          );

          uploadedParts.push({
            number: chunk.number,
            etag: result.etag,
          });

          onChunkComplete({
            number: chunk.number,
            total: chunkCount,
            etag: result.etag,
          });

          // 更新进度
          const progress = (uploadedParts.length / chunkCount) * 100;
          onProgress(progress);

          return result;
        } catch (error) {
          console.error(`分片 ${chunk.number} 上传失败:`, error);

          // 重试逻辑
          if (chunk.retries < this.retryCount) {
            chunk.retries++;
            console.log(`分片 ${chunk.number} 第 ${chunk.retries} 次重试`);
            return uploadChunk(chunk);
          } else {
            failedChunks.push(chunk.number);
            throw error;
          }
        }
      };

      // 执行并发上传
      while (queue.length > 0 || running.length > 0) {
        // 填充并发队列
        while (running.length < this.concurrent && queue.length > 0) {
          const chunk = queue.shift();
          const promise = uploadChunk(chunk).finally(() => {
            running.splice(running.indexOf(promise), 1);
          });
          running.push(promise);
        }

        // 等待任一任务完成
        if (running.length > 0) {
          await Promise.race(running);
        }
      }

      // 检查是否有失败的分片
      if (failedChunks.length > 0) {
        throw new Error(`以下分片上传失败: ${failedChunks.join(', ')}`);
      }

      // 排序分片
      uploadedParts.sort((a, b) => a.number - b.number);

      // 完成分片上传
      const result = await this.uploader.client.completeMultipartUpload(
        fileName,
        uploadId,
        uploadedParts,
      );

      onComplete(result);
      return result;
    } catch (error) {
      console.error('分片上传失败:', error);
      throw error;
    }
  }

  /**
   * 取消上传
   */
  cancel() {
    // 实现取消逻辑
  }
}

export default ChunkUploader;
```

### 6.3 断点续传

```js
// src/utils/resumableUploader.js
/**
 * 断点续传实现
 */
class ResumableUploader {
  constructor(uploader) {
    this.uploader = uploader;
    this.dbName = 'uploadCache';
    this.storeName = 'uploads';
    this.db = null;
  }

  /**
   * 初始化IndexedDB
   */
  async initDB() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        if (!db.objectStoreNames.contains(this.storeName)) {
          const store = db.createObjectStore(this.storeName, { keyPath: 'id' });
          store.createIndex('fileId', 'fileId', { unique: true });
          store.createIndex('timestamp', 'timestamp');
        }
      };
    });
  }

  /**
   * 保存上传进度
   */
  async saveProgress(fileId, data) {
    if (!this.db) await this.initDB();

    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);

      const request = store.put({
        id: fileId,
        fileId,
        ...data,
        timestamp: Date.now(),
      });

      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }

  /**
   * 获取上传进度
   */
  async getProgress(fileId) {
    if (!this.db) await this.initDB();

    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readonly');
      const store = transaction.objectStore(this.storeName);
      const index = store.index('fileId');

      const request = index.get(fileId);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        resolve(request.result || null);
      };
    });
  }

  /**
   * 删除上传进度
   */
  async deleteProgress(fileId) {
    if (!this.db) await this.initDB();

    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const index = store.index('fileId');

      // 先获取主键
      const getRequest = index.getKey(fileId);

      getRequest.onerror = () => reject(getRequest.error);
      getRequest.onsuccess = () => {
        if (getRequest.result) {
          const deleteRequest = store.delete(getRequest.result);
          deleteRequest.onerror = () => reject(deleteRequest.error);
          deleteRequest.onsuccess = () => resolve();
        } else {
          resolve();
        }
      };
    });
  }

  /**
   * 清理过期缓存（7天前）
   */
  async cleanExpired() {
    if (!this.db) await this.initDB();

    const sevenDaysAgo = Date.now() - 7 * 24 * 60 * 60 * 1000;

    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const index = store.index('timestamp');

      const range = IDBKeyRange.upperBound(sevenDaysAgo);
      const request = index.openCursor(range);

      request.onerror = () => reject(request.error);
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        if (cursor) {
          store.delete(cursor.primaryKey);
          cursor.continue();
        } else {
          resolve();
        }
      };
    });
  }
}

export default ResumableUploader;
```

## 七、最佳实践指南

### 7.1 错误处理

```js
// src/utils/errorHandler.js
/**
 * 错误类型枚举
 */
export const UploadErrorType = {
  NETWORK: 'network_error',
  TIMEOUT: 'timeout_error',
  AUTH: 'auth_error',
  SIZE: 'size_error',
  TYPE: 'type_error',
  SERVER: 'server_error',
  UNKNOWN: 'unknown_error',
};

/**
 * 错误处理器
 */
export class ErrorHandler {
  /**
   * 处理OSS错误
   */
  static handleOSSError(error) {
    console.error('OSS错误详情:', error);

    // 网络错误
    if (error.code === 'ConnectionTimeout' || error.name === 'TimeoutError') {
      return {
        type: UploadErrorType.TIMEOUT,
        message: '上传超时，请检查网络后重试',
        retryable: true,
      };
    }

    // 认证错误
    if (error.code === 'AccessDenied' || error.code === 'InvalidAccessKeyId') {
      return {
        type: UploadErrorType.AUTH,
        message: '权限验证失败，请重新登录',
        retryable: true,
      };
    }

    // 服务器错误
    if (error.statusCode >= 500) {
      return {
        type: UploadErrorType.SERVER,
        message: '服务器错误，请稍后重试',
        retryable: true,
      };
    }

    // 请求错误
    if (error.statusCode === 400) {
      return {
        type: UploadErrorType.NETWORK,
        message: '请求参数错误',
        retryable: false,
      };
    }

    // 未知错误
    return {
      type: UploadErrorType.UNKNOWN,
      message: error.message || '上传失败，请重试',
      retryable: false,
    };
  }

  /**
   * 显示错误提示
   */
  static showError(errorInfo) {
    // 可以根据环境使用不同的提示方式
    if (typeof window !== 'undefined') {
      alert(errorInfo.message);
    }

    // 可以在这里集成第三方错误监控服务
    // Sentry.captureException(errorInfo);
  }

  /**
   * 重试逻辑
   */
  static async retry(fn, maxRetries = 3, delay = 1000) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn();
      } catch (error) {
        const errorInfo = this.handleOSSError(error);

        if (!errorInfo.retryable || i === maxRetries - 1) {
          throw error;
        }

        console.log(`第 ${i + 1} 次重试，等待 ${delay}ms`);
        await new Promise((resolve) => setTimeout(resolve, delay));
        delay *= 2; // 指数退避
      }
    }
  }
}
```

### 7.2 性能优化

```js
// src/utils/performanceOptimizer.js
/**
 * 性能优化工具
 */
export class PerformanceOptimizer {
  constructor() {
    this.workers = new Set();
  }

  /**
   * 使用Web Worker处理大文件
   */
  createUploadWorker() {
    const workerCode = `
      self.onmessage = async function(e) {
        const { file, options } = e.data;
        
        // 分片处理
        const chunkSize = options.chunkSize || 1024 * 1024;
        const chunks = [];
        
        for (let start = 0; start < file.size; start += chunkSize) {
          const end = Math.min(start + chunkSize, file.size);
          chunks.push(file.slice(start, end));
        }
        
        // 返回分片信息
        self.postMessage({
          type: 'chunks',
          data: chunks,
          total: chunks.length
        });
      };
    `;

    const blob = new Blob([workerCode], { type: 'application/javascript' });
    const workerUrl = URL.createObjectURL(blob);
    const worker = new Worker(workerUrl);

    this.workers.add(worker);
    return worker;
  }

  /**
   * 预加载连接
   */
  preconnect(urls) {
    urls.forEach((url) => {
      const link = document.createElement('link');
      link.rel = 'preconnect';
      link.href = url;
      link.crossOrigin = 'anonymous';
      document.head.appendChild(link);
    });
  }

  /**
   * 缓存上传进度到LocalStorage
   */
  cacheProgress(fileId, progress) {
    const cacheKey = `upload_${fileId}`;
    const cacheData = {
      progress,
      timestamp: Date.now(),
      fileId,
    };

    localStorage.setItem(cacheKey, JSON.stringify(cacheData));
  }

  /**
   * 获取缓存的进度
   */
  getCachedProgress(fileId) {
    const cacheKey = `upload_${fileId}`;
    const cached = localStorage.getItem(cacheKey);

    if (cached) {
      const data = JSON.parse(cached);
      // 检查缓存是否过期（24小时）
      if (Date.now() - data.timestamp < 24 * 60 * 60 * 1000) {
        return data.progress;
      }
    }

    return null;
  }

  /**
   * 清理worker
   */
  cleanup() {
    this.workers.forEach((worker) => {
      worker.terminate();
    });
    this.workers.clear();
  }
}
```

## 八、常见问题排查

### 8.1 错误码对照表

| 错误码 | 错误信息 | 可能原因 | 解决方案 |

|--------|----------|----------|----------|

| 403 | AccessDenied | STS凭证过期 | 刷新Token或重新获取 |
| 403 | InvalidAccessKeyId | AccessKey错误 | 检查STS凭证配置 |
| 404 | NoSuchBucket | Bucket不存在 | 检查Bucket名称 |
| 404 | NoSuchKey | 文件不存在 | 检查文件路径 |
| 408 | RequestTimeout | 请求超时 | 调整超时设置，检查网络 |
| 409 | BucketAlreadyExists | Bucket已存在 | 更换Bucket名称 |
| 411 | MissingContentLength | 缺少Content-Length | 检查请求头设置 |
| 413 | EntityTooLarge | 文件太大 | 使用分片上传 |
| 500 | InternalError | 服务器内部错误 | 重试或联系技术支持 |

## 九、总结

### 9.1 核心要点总结

#### 安全性优先

- 永远不要在前端暴露AccessKey
- 使用STS临时凭证
- 设置合理的权限和有效期

#### 用户体验

- 提供上传进度反馈
- 支持拖拽上传
- 文件预览功能
- 友好的错误提示

#### 性能优化

- 大文件分片上传
- 图片压缩处理
- 并发控制
- 断点续传

#### 可维护性

- 模块化设计
- 完整的错误处理
- 日志记录
- 监控告警
