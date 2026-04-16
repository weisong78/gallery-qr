# Gallery QR — 扫码展厅

## 概念与愿景

一个精致的数字展厅工具，创作者上传视频/图片+文字介绍，系统为每个作品生成独立链接和二维码，观众扫码即可沉浸式浏览。适合展览图录、作品集、产品手册、活动回顾等场景。

视觉风格：**Gallery Noir** — 像走进一间高端艺术展厅。深色背景衬出作品的色彩，安静、沉浸、专注。字体优雅克制，间距宽松，像策展人的精心布置。

## 设计语言

### 美学方向
艺术画廊 / 暗色沉浸式 / 策展人质感

### 色彩
```
--bg:        #0c0c0e       深墨色背景
--surface:   #161618       卡片/面板
--border:    #2a2a2e       微妙的边框
--text-1:    #f0ede8       主文字（暖白）
--text-2:    #8a8780       次要文字（灰）
--accent:    #c9a96e       金色点缀（画廊黄铜质感）
--accent-2:  #5b8a6e       青绿（成功/确认）
--danger:    #a65b5b       暖红（错误）
```

### 字体
- 展示字体：`Playfair Display`（衬线，经典优雅）— 标题、标签
- 正文字体：`Crimson Pro`（衬线，适合阅读）— 描述文字
- 辅助字体：`JetBrains Mono`（等宽）— 链接、代码

### 动效
- 页面加载：staggered fade-up（图片墙逐个浮现，间隔 80ms）
- 卡片 hover：微上浮 + 阴影加深，200ms ease-out
- 二维码弹窗：从中心 scale 弹出，backdrop 淡入
- 模态框打开：图片从点击位置放大展开

## 布局与结构

### 双模式界面

**🔧 管理模式（`?mode=admin`）**
- 顶部导航：Logo + 模式切换 + 存储状态指示
- 主内容：已上传作品网格（3列） + 底部浮动上传区
- 每个作品卡片：缩略图 + 标题 + 类型标签 + 操作按钮（查看二维码 / 删除）

**👁 查看模式（`?view=<id>`）**
- 全屏沉浸式：纯作品展示，无多余UI
- 顶部：简洁返回按钮 + 作品标题
- 中央：图片/视频（自适应大小，最大 90vw / 80vh）
- 底部：文字介绍区（标题 + 描述）
- 底部固定：二维码入口按钮

### 响应式
- 移动端：单列布局，点击全屏查看
- 桌面端：网格布局，hover 效果

## 功能与交互

### 上传功能
- 支持拖拽上传 / 点击选择
- 支持格式：`.jpg .png .gif .webp .mp4 .webm .mov`
- 文件本地预览（图片即时缩略，video 提取首帧）
- 上传后填写标题 + 描述（可不填，有默认值）
- 存储到 localStorage（图片/视频转为 base64，小文件适用）

### 二维码生成
- 为每个作品生成唯一 URL：`?view=<nanoid>`
- 使用 qrcode.js 生成 QR 码图
- 点击卡片二维码按钮 → 弹出模态框显示大尺寸二维码
- 支持下载二维码 PNG

### 查看模式交互
- 图片：点击放大 lightbox
- 视频：原生 video 控制条，支持全屏
- 键盘支持：Escape 关闭 lightbox，← → 切换（如有多图）

### 空状态
- 无作品时：优雅的空白状态 + 引导文案「上传你的第一件作品」

### 错误处理
- 文件过大（>20MB）→ 提示并拒绝
- 不支持格式 → 友好提示
- localStorage 满 → 警告并建议清理

## 组件清单

### 1. NavBar
- 左：Logo 文字（Playfair Display）
- 右：admin 切换标签 / 存储用量条
- 状态：固定顶部，backdrop blur

### 2. MediaCard（作品卡片）
- 状态：默认 / hover（上浮+阴影）/ loading（骨架屏）
- 内容：媒体预览 + 标题 + 类型角标 + 操作按钮组
- hover 显示操作按钮（查看二维码、删除）

### 3. UploadZone（上传区）
- 状态：默认（虚线边框+图标）/ dragover（边框变金色+放大）/ 上传中（进度动画）
- 点击打开文件选择器

### 4. QRModal（二维码弹窗）
- 中央大二维码 + 链接文字 + 下载按钮
- 点击 backdrop 关闭

### 5. ViewerFull（查看模式）
- 全屏媒体展示
- 图片 lightbox 支持

### 6. EmptyState
- 居中插图 + 引导文案

## 技术方案

### 单文件 HTML
- 所有代码（HTML + CSS + JS）打包在 `index.html`
- 外部依赖（CDN）：
  - Google Fonts（Playfair Display, Crimson Pro, JetBrains Mono）
  - qrcode.js（`https://cdn.jsdelivr.net/npm/qrcode@1.5.3/build/qrcode.min.js`）
  - nanoid（`https://cdn.jsdelivr.net/npm/nanoid@3.3.7/index.browser.js`）

### 数据模型
```js
// localStorage key: "gallery_qr_items"
{
  id: "abc123",          // nanoid
  type: "image" | "video",
  mediaData: "data:image/...", // base64 或 blob URL
  title: "作品标题",
  description: "文字介绍...",
  createdAt: 1700000000000,
  thumbnail: "data:image/...", // 视频首帧
}
```

### 路由逻辑
- `?mode=admin` → 管理模式（默认）
- `?view=<id>` → 查看指定作品
- `?mode=admin&view=<id>` → 管理员预览（同时显示二维码）

### 文件大小限制
- 图片：10MB
- 视频：50MB
- 超出提示，不存储
