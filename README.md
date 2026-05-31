# 读图训练项目 - 设计文档

> 创建时间：2026-05-19
> 版本：v1_initial（功能大致完善）

---

## 项目概述

Brooks Price Action 蜡烛图读图训练工具。核心功能是黑幕逐渐揭示，让用户在没有整体预览的情况下逐步揭示图片，训练自己的价格行为识别能力。

**线上地址：** https://bigboom1.github.io/chart-reading-trainer/viewer/

---

## 核心功能

1. **黑幕揭示机制** — 黑色遮罩从左侧裁剪，图片从左往右逐渐露出
2. **分割数控制** — 可设置分割数（默认32，最大128）
3. **揭示幅度** — 每步揭示量可调（默认1帧）
4. **版本切换** — 支持 v2024.3.1（121页）和 v2024.4.1（102页）
5. **翻页导航** — 上一页/下一页/重置
6. **快捷键** — →/Space 揭示，← 后退，↑↓ 翻页，R 重置

---

## 技术实现

### clip-path 揭示原理

```
inset(上 右 下 左)

revealed = 0%   → inset(0 0 0 0%)   → 全遮罩，图片不可见
revealed = 50%  → inset(0 0 0 50%)  → 左侧50%裁掉，右侧50%图片露出
revealed = 100% → inset(0 0 0 100%) → 完全裁掉，图片完整可见
```

**关键**：第四个参数控制左侧裁剪量，revealed 值越大，左侧裁得越多，右侧露出的图片越多。

### 文件结构

```
读图训练/
├── viewer/
│   ├── index.html              # 当前线上版本
│   └── index.v1_initial.html  # v1 备份（功能完善版本）
├── assets/
│   └── pages-sm/
│       ├── v2024.3.1/          # 降级后的图片（121页）
│       └── v2024.4.1/          # 降级后的图片（102页）
├── README.md                   # 本文件
└── viewer/index.html           # 当前工作版本
```

### 关键代码片段

```javascript
// 揭示逻辑核心
function updateReveal() {
  maskEl.style.clipPath = `inset(0 0 0 ${currentReveal}%)`;
  pBar.value = currentReveal;
  pText.textContent = `${Math.round(currentReveal)}%`;
}

function doReveal() {
  if (currentReveal >= 100) return;
  currentReveal = Math.min(100, currentReveal + (100 / slices) * step);
  updateReveal();
}
```

---

## 历史记录

### 2026-05-19（v1 开发过程）

| 问题 | 修改 | 结果 |
|------|------|------|
| clip-path 用错参数顺序 | inset(上 右 下 左) 第四参数=左侧裁剪 | 揭示方向从右往左 |
| 揭示方向反了 | 改为 inset(0 0 0 N%) 从左裁剪 | 图片从左往右露出 ✅ |
| 去掉点击提示 | 移除 click-hint 元素和相关代码 | 界面更简洁 |

### 修复过程（用户反馈）

- 最初：revealed=0 → clipPath=0%（遮罩被裁掉，图片可见）❌
- 第1次：改为 revealed=0 → clipPath=100%（全遮罩）❌（方向反了）
- 第2次：改为 inset(N% 0 0 0) 从上方裁 ❌（变成从上往下）
- 第3次：改为 inset(0 N% 0 0) 从右裁 ❌（从右往左）
- 第4次：改为 inset(0 0 0 N%) 从左裁 ✅（从左往右）

---

## 后续可完善方向

1. **揭示方向** — 用户可能还想要更多方向选择（从右往左、从上往下、从中心向外）
2. **揭示动画** — 加个平滑过渡动画，而不是跳跃式
3. **帧数预设** — 常见分割数快捷按钮（16/32/64/128）
4. **进度记忆** — 记住上次看到第几页
5. **全屏模式** — 隐藏顶部/底部栏，全屏练习
6. **随机出题** — 随机跳到某一页，增强训练效果
7. **移动端优化** — 大屏幕手机上更好的触摸体验

---

## 相关资源

- GitHub 仓库：https://github.com/bigboom1/chart-reading-trainer
- 图片版本：v2024.3.1（121页）+ v2024.4.1（102页）
- 原图降级：194MB → 45MB（通过降低分辨率实现）
- Token：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（注意：曾泄露密码，需关注安全）

---

*本文件是项目的长期记忆，确保后续开发时有足够的上下文信息。*