---
title: Vue2+ElementUI+compressor.js实现高质量图片压缩
tags:
  - 图片压缩
  - vue2
  - js
author: Eric
date: 2026-2-11
comments: false
categories: 图片压缩
cover: /img/v1.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

## Vue2+ElementUI+compressor.js实现高质量图片压缩 DEMO

```js
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vue2图片压缩上传 - compressor.js</title>
  <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
  <script src="https://unpkg.com/element-ui/lib/index.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/compressorjs@1.1.1/dist/compressor.min.js"></script>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB', Arial, sans-serif;
      background: linear-gradient(135deg, #f5f7fa 0%, #e4edf5 100%);
      min-height: 100vh;
      padding: 20px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .container {
      max-width: 1000px;
      width: 100%;
      background: white;
      border-radius: 12px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
      overflow: hidden;
    }
    .header {
      background: linear-gradient(90deg, #409EFF 0%, #64b5f6 100%);
      color: white;
      padding: 25px 30px;
      text-align: center;
    }
    .header h1 {
      font-size: 24px;
      margin-bottom: 8px;
    }
    .header p {
      opacity: 0.9;
      font-size: 15px;
    }
    .content {
      padding: 30px;
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 30px;
    }
    @media (max-width: 768px) {
      .content {
        grid-template-columns: 1fr;
      }
    }
    .upload-section {
      background: #f8fafc;
      border-radius: 10px;
      padding: 25px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
    }
    .preview-section {
      background: #f8fafc;
      border-radius: 10px;
      padding: 25px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
    }
    .section-title {
      font-size: 18px;
      color: #409EFF;
      margin-bottom: 20px;
      padding-bottom: 12px;
      border-bottom: 1px solid #ebeef5;
      display: flex;
      align-items: center;
    }
    .section-title i {
      margin-right: 10px;
      font-size: 20px;
    }
    .upload-area {
      border: 2px dashed #dcdfe6;
      border-radius: 8px;
      padding: 30px 20px;
      text-align: center;
      background: white;
      transition: all 0.3s;
      cursor: pointer;
      margin-bottom: 20px;
    }
    .upload-area:hover {
      border-color: #409EFF;
      background: #f0f7ff;
    }
    .upload-icon {
      font-size: 48px;
      color: #409EFF;
      margin-bottom: 15px;
    }
    .upload-text {
      color: #606266;
      margin-bottom: 15px;
    }
    .upload-text h3 {
      margin-bottom: 8px;
      font-weight: 600;
    }
    .preview-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 20px;
    }
    .preview-box {
      width: 100%;
      background: white;
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
    }
    .preview-title {
      background: #f5f7fa;
      padding: 10px 15px;
      font-weight: 600;
      color: #409EFF;
      border-bottom: 1px solid #ebeef5;
    }
    .preview-image {
      display: block;
      max-width: 100%;
      max-height: 200px;
      margin: 0 auto;
      padding: 15px;
    }
    .file-info {
      padding: 15px;
      background: #f9fbfd;
      border-top: 1px solid #ebeef5;
    }
    .info-row {
      display: flex;
      justify-content: space-between;
      margin-bottom: 8px;
      font-size: 14px;
    }
    .info-label {
      color: #606266;
      font-weight: 500;
    }
    .info-value {
      color: #409EFF;
      font-weight: 600;
    }
    .compression-ratio {
      text-align: center;
      font-size: 16px;
      font-weight: 600;
      color: #67C23A;
      margin: 10px 0;
    }
    .compression-ratio.low {
      color: #E6A23C;
    }
    .compression-ratio.high {
      color: #F56C6C;
    }
    .controls {
      margin-top: 25px;
    }
    .slider-control {
      margin-bottom: 20px;
    }
    .slider-label {
      display: flex;
      justify-content: space-between;
      margin-bottom: 8px;
      font-size: 14px;
      color: #606266;
    }
    .slider-value {
      font-weight: 600;
      color: #409EFF;
    }
    .actions {
      display: flex;
      gap: 15px;
      margin-top: 20px;
    }
    .status {
      margin-top: 20px;
      padding: 15px;
      border-radius: 6px;
      background: #f0f9eb;
      color: #67C23A;
      text-align: center;
      display: none;
    }
    .status.error {
      background: #fef0f0;
      color: #F56C6C;
    }
    .status.active {
      display: block;
    }
    .footer {
      text-align: center;
      padding: 25px;
      background: #f8fafc;
      color: #909399;
      font-size: 14px;
      border-top: 1px solid #ebeef5;
    }
  </style>
</head>
<body>
  <div id="app">
    <div class="container">
      <div class="header">
        <h1>Vue2 + Element UI 图片压缩上传</h1>
        <p>使用compressor.js实现高质量图片压缩</p>
      </div>
      
      <div class="content">
        <div class="upload-section">
          <div class="section-title">
            <i class="el-icon-upload"></i>
            <span>上传图片</span>
          </div>
          
          <el-upload
            class="upload-area"
            drag
            action="#"
            :auto-upload="false"
            :on-change="handleFileChange"
            :show-file-list="false"
            accept="image/*"
          >
            <div>
              <div class="upload-icon">
                <i class="el-icon-upload"></i>
              </div>
              <div class="upload-text">
                <h3>点击或拖拽图片到此处上传</h3>
                <p>支持 JPG/PNG/GIF 格式，最大 10MB</p>
              </div>
              <el-button size="medium" type="primary">选择图片</el-button>
            </div>
          </el-upload>
          
          <div class="controls">
            <div class="slider-control">
              <div class="slider-label">
                <span>压缩质量: {{ quality }}%</span>
                <span class="slider-value">(值越小文件越小)</span>
              </div>
              <el-slider v-model="quality" :min="30" :max="95" :step="5"></el-slider>
            </div>
            
            <div class="slider-control">
              <div class="slider-label">
                <span>最大宽度: {{ maxWidth }}px</span>
                <span class="slider-value">(0表示保持原尺寸)</span>
              </div>
              <el-slider v-model="maxWidth" :min="0" :max="2000" :step="100"></el-slider>
            </div>
            
            <div class="actions">
              <el-button 
                type="primary" 
                :disabled="!file" 
                @click="compressImage"
                :loading="compressing"
              >
                <i class="el-icon-refresh"></i> 压缩图片
              </el-button>
              <el-button 
                type="success" 
                :disabled="!compressedFile" 
                @click="uploadImage"
                :loading="uploading"
              >
                <i class="el-icon-upload"></i> 上传图片
              </el-button>
              <el-button 
                type="info" 
                @click="reset"
              >
                <i class="el-icon-delete"></i> 重置
              </el-button>
            </div>
          </div>
          
          <div class="status" :class="{ active: statusMessage, error: statusType === 'error' }">
            <i :class="statusType === 'error' ? 'el-icon-error' : 'el-icon-success'"></i>
            {{ statusMessage }}
          </div>
        </div>
        
        <div class="preview-section">
          <div class="section-title">
            <i class="el-icon-picture"></i>
            <span>图片预览</span>
          </div>
          
          <div class="preview-container">
            <div class="preview-box" v-if="file">
              <div class="preview-title">原始图片</div>
              <img :src="originalUrl" class="preview-image" alt="原始图片">
              <div class="file-info">
                <div class="info-row">
                  <span class="info-label">文件名:</span>
                  <span class="info-value">{{ file.name }}</span>
                </div>
                <div class="info-row">
                  <span class="info-label">文件类型:</span>
                  <span class="info-value">{{ file.type }}</span>
                </div>
                <div class="info-row">
                  <span class="info-label">文件大小:</span>
                  <span class="info-value">{{ formatFileSize(file.size) }}</span>
                </div>
              </div>
            </div>
            
            <div class="preview-box" v-if="compressedFile">
              <div class="preview-title">压缩后图片</div>
              <img :src="compressedUrl" class="preview-image" alt="压缩后图片">
              <div class="file-info">
                <div class="info-row">
                  <span class="info-label">压缩质量:</span>
                  <span class="info-value">{{ quality }}%</span>
                </div>
                <div class="info-row">
                  <span class="info-label">文件大小:</span>
                  <span class="info-value">{{ formatFileSize(compressedFile.size) }}</span>
                </div>
                <div class="compression-ratio" :class="{
                  'low': compressionRatio < 50, 
                  'high': compressionRatio > 80
                }">
                  压缩率: {{ compressionRatio }}%
                </div>
              </div>
            </div>
            
            <div v-if="!file" class="no-preview">
              <i class="el-icon-picture-outline" style="font-size: 60px; color: #dcdfe6;"></i>
              <p style="margin-top: 15px; color: #909399;">请上传图片查看预览</p>
            </div>
          </div>
        </div>
      </div>
      
      <div class="footer">
        <p>Vue2 + Element UI | compressor.js 图片压缩上传示例</p>
      </div>
    </div>
  </div>

  <script>
    new Vue({
      el: '#app',
      data() {
        return {
          file: null,
          compressedFile: null,
          originalUrl: '',
          compressedUrl: '',
          quality: 80,
          maxWidth: 1200,
          compressing: false,
          uploading: false,
          statusMessage: '',
          statusType: 'success'
        }
      },
      computed: {
        compressionRatio() {
          if (!this.file || !this.compressedFile) return 0;
          const ratio = (1 - this.compressedFile.size / this.file.size) * 100;
          return Math.round(ratio);
        }
      },
      methods: {
        // 处理文件选择
        handleFileChange(file) {
          if (!file) return;
          
          // 检查文件类型
          if (!file.raw.type.startsWith('image/')) {
            this.showStatus('请选择图片文件', 'error');
            return;
          }
          
          // 检查文件大小 (10MB限制)
          if (file.size > 10 * 1024 * 1024) {
            this.showStatus('文件大小不能超过10MB', 'error');
            return;
          }
          
          this.file = file.raw;
          this.compressedFile = null;
          this.compressedUrl = '';
          this.originalUrl = URL.createObjectURL(this.file);
          this.showStatus('图片已选择，请调整压缩参数后点击"压缩图片"');
        },
        
        // 压缩图片
        compressImage() {
          if (!this.file) return;
          
          this.compressing = true;
          this.showStatus('图片压缩中...');
          
          // 使用compressor.js进行图片压缩
          new Compressor(this.file, {
            quality: this.quality / 100,
            width: this.maxWidth || undefined,
            success: (result) => {
              this.compressedFile = result;
              this.compressedUrl = URL.createObjectURL(result);
              this.compressing = false;
              
              const originalSize = this.formatFileSize(this.file.size);
              const compressedSize = this.formatFileSize(result.size);
              this.showStatus(`压缩成功！${originalSize} → ${compressedSize}，压缩率: ${this.compressionRatio}%`);
            },
            error: (err) => {
              console.error('压缩失败:', err);
              this.compressing = false;
              this.showStatus('图片压缩失败: ' + err.message, 'error');
            }
          });
        },
        
        // 上传图片
        uploadImage() {
          if (!this.compressedFile) return;
          
          this.uploading = true;
          this.showStatus('上传中，请稍候...');
          
          // 模拟上传请求
          setTimeout(() => {
            this.uploading = false;
            this.showStatus('上传成功！图片已保存到服务器');
            
            // 在实际项目中，这里应该发送真实的API请求
            /*
            const formData = new FormData();
            formData.append('file', this.compressedFile);
            
            axios.post('/api/upload', formData, {
              headers: { 'Content-Type': 'multipart/form-data' }
            })
            .then(response => {
              this.uploading = false;
              this.showStatus('上传成功！URL: ' + response.data.url);
            })
            .catch(error => {
              this.uploading = false;
              this.showStatus('上传失败: ' + error.message, 'error');
            });
            */
          }, 1500);
        },
        
        // 重置所有状态
        reset() {
          this.file = null;
          this.compressedFile = null;
          this.originalUrl = '';
          this.compressedUrl = '';
          this.statusMessage = '';
        },
        
        // 显示状态消息
        showStatus(message, type = 'success') {
          this.statusMessage = message;
          this.statusType = type;
          
          // 5秒后自动清除消息
          setTimeout(() => {
            if (this.statusMessage === message) {
              this.statusMessage = '';
            }
          }, 5000);
        },
        
        // 格式化文件大小
        formatFileSize(bytes) {
          if (bytes === 0) return '0 Bytes';
          const k = 1024;
          const sizes = ['Bytes', 'KB', 'MB', 'GB'];
          const i = Math.floor(Math.log(bytes) / Math.log(k));
          return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
        }
      }
    });
  </script>
</body>
</html>
```
