# 交互式滤波器频率响应页面 · 项目上下文

> 迁移用，包含完整项目背景、功能清单、技术实现和待办事项。

---

## 项目概述

从零学习数字滤波器，以 AD7768-1 和 ADS127L01 的滤波器设计为学习对象，构建交互式 HTML 页面辅助理解。

- **GitHub Pages**：https://jiexwang.github.io/filter_response/
- **GitHub 仓库**：https://github.com/jiexwang/filter_response
- **本地路径**：`E:\work\AS127L01_code\filter_response\`

---

## 文件结构

```
filter_response/
├── index.html              ← 主文件（单文件，所有 CSS/JS 都在内）
├── deploy/
│   └── index.html          ← GitHub Pages 部署副本（与 index.html 同步）
└── CONTEXT.md              ← 本文件
```

---

## 已完成的功能模块

### 1. Sinc^N 频率响应（主 tab）

**基础功能**：
- 可调参数：级数 N（1-10）、抽取率 D（2-1000）、f_MOD（kHz）、ODR（sps）
- 两种视图：通带视图（0~ODR）+ 全范围视图（0~f_MOD/2），上下排列同时显示
- 关键频点衰减表（f/ODR = 0, 0.1, 0.2, 0.3, 0.4, 0.433, 0.5）
- 公式渲染（Markdown 风格）
- 游标悬停显示实时衰减值
- 补偿目标曲线（1/Sinc^N）
- 零点标注和旁瓣峰值标注
- 导出 PNG
- 浅色/深色主题切换

**级联 Sinc 模式**：
- 可启用多级 sinc 级联
- 每级独立设置级数 N 和抽取率 D
- 默认配置：Sinc5(D=32) + Sinc1(D=4)，模拟 ADS127L01 Low-Latency 模式
- 各级用不同颜色虚线绘制，总响应用实线
- 总抽取率自动计算
- 图例自动更新
- 游标显示各级单独衰减 + 级联总衰减

### 2. FIR 偶对称与线性相位（第二 tab）

**功能**：
- 旋转向量动画：可视化理解为什么对称 FIR = 线性相位
- 可编辑系数输入 + 预设按钮（[1,2,1], [1,2,0.5], [1,-1], [1,1,1,1], [1,3,3,1]）
- 播放/暂停/重置动画，ω 从 0 扫到 2π
- 幅频 + 相频响应图（带相位解包裹）
- 动画轨迹可视化
- Markdown 风格公式渲染（H(ω) = ...）
- 对称性自动检测（偶对称/奇对称/非对称）
- 知识检验题目

---

## 关键技术实现

### Sinc 计算

```javascript
function sincN(f, N, D, fMOD) {
  const w = 2 * Math.PI * f / fMOD;
  const den = Math.sin(w / 2);
  if (Math.abs(den) < 1e-14) return 1.0;
  return Math.pow(Math.abs(Math.sin(D * w / 2) / (D * den)), N);
}
```

### 级联总响应

各级 dB 值相加（等价于线性域相乘）：

```javascript
function cascadedSinc_linear(f) {
  return sincStages.reduce((prod, s) => prod * sincN(f, s.N, s.D, P.fMOD), 1);
}
```

### Canvas 架构

两个独立 canvas（passbandCanvas + fullrangeCanvas），各有独立坐标映射和鼠标交互。

### 相位解包裹

消除 ±π 跳变，使相频曲线连续：

```javascript
for (let i = 1; i <= steps; i++) {
  let diff = pts[i].phase - pts[i - 1].phase;
  while (diff > Math.PI) diff -= 2 * Math.PI;
  while (diff < -Math.PI) diff += 2 * Math.PI;
  pts[i].phase = pts[i - 1].phase + diff;
}
```

### 颜色方案

```css
--sinc:#58a6ff (蓝)      /* 总响应/单级 sinc */
--comp:#3fb950 (绿)      /* 0 dB 参考线 */
--total:#f0c674 (黄)     /* 级联总响应 */
--target:#ff5555 (红)    /* 补偿目标 */
```

级联各级颜色：`['#58a6ff', '#d2a8ff', '#79c0ff', '#ffa657', '#7ee787', '#ff7b72']`

---

## 学习内容记录

### 已覆盖的知识点

1. **FIR 偶对称 → 线性相位**：旋量图直觉理解，对称系数使虚部抵消
2. **AD7768-1 架构**：Sinc⁶ → 补偿 FIR → 主 FIR
3. **ADS127L01 架构**：
   - Wideband 模式：Sinc5 → HB1 → Comp → FIR+HB2
   - Low-Latency 模式：Sinc5(D=32 固定) + Sinc1(D=OSR 可变)
4. **Low-Latency 建立时间公式**：`Settling(tCLK) = 160 + OSR`（已写入手册 md 文件）
5. **Half-band 滤波器**：为什么 AD7768-1 不需要（Sinc⁶ 衰减足够），ADS127L01 需要（Sinc⁵ 不够）
6. **Sinc 阶数影响**：每多一级级联，旁瓣衰减增加约 20dB

### 已修改的文档

- `E:\work\AS127L01_code\doc\ads127l01.md`：Table 3 后插入了 Low-Latency 建立时间公式分析（带蓝色背景框 div）

---

## 相关文件

| 文件 | 说明 |
|------|------|
| `E:\work\AS127L01_code\filter_response\index.html` | 交互页面主文件 |
| `E:\work\AS127L01_code\doc\ads127l01.md` | ADS127L01 手册（含 Table 3 分析） |
| `E:\work\AS127L01_code\rtl_old\filter_top.v` | ADS127L01 滤波器链 RTL |
| `E:\work\AS127L01_code\doc\ads127l01.pdf` | ADS127L01 原始手册 PDF |
| `E:\work\AS127L01_code\doc\zhcacg1a.pdf` | TI 应用笔记（Δ-Σ ADC 滤波器类型） |
| `E:\work\filt_7768\` | 原始项目目录（已迁移） |

---

## 待办事项

- [ ] 推送到 GitHub（网络问题，需手动或等网络恢复）
- [ ] 如需推到新仓库，运行：
  ```bash
  cd E:\work\AS127L01_code\filter_response
  git remote add origin https://github.com/jiexwang/AS127L01_filter_response.git
  git push -u origin master
  ```

---

## 用户偏好

- 用户是数字滤波器初学者，喜欢从基本原理出发的"为什么"式讲解
- 偏好时域直觉理解，不喜欢纯频域数学推导
- 喜欢交互式可视化辅助理解
- 会问很基础但很关键的问题（如"为什么偶对称=线性相位"）
