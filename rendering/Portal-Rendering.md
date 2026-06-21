---
tags: [rendering, portal, render-target, technique]
aliases: [传送门渲染, Portal]
---

# Portal Rendering（传送门渲染）

传送门是 [[Render-Target|Render Target]] 的经典应用——把另一个摄像机的画面实时贴到一个面上。

---

## 核心思路

```
不是两个世界，是同一个世界渲染了两次

同一个场景里：

  房间A ─────────────────── 房间B

  📷 主摄像机（玩家）        📷 隐藏摄像机
  在这边看                   在这边看
```

传送门的"洞"= 一张实时更新的 RT 贴在一个平面上。

---

## 两个 Pass 的结构

```
Pass 1: Portal Camera 渲染出口视角
  摄像机：出口处的隐藏摄像机
  输出：Portal RT

Pass 2: 渲染主场景
  摄像机：玩家
  传送门那个面：采样 Portal RT 作为贴图
  输出：Backbuffer
```

---

## 类比

就像**监控摄像头屏幕**：

```
监控摄像头（出口的隐藏摄像机）
       ↓ 拍摄
监控画面（Portal RT）
       ↓ 显示在
监控屏幕（传送门这个面）
```

---

## Portal Camera 怎么算位置？

玩家相对于入口的偏移，映射到出口：

```
玩家往左走 → Portal Camera 也往左走
玩家靠近入口 → Portal Camera 靠近出口
玩家进入传送门 → 坐标从入口传送到出口
```

```cpp
// Portal Camera 的变换 = 出口变换 × 入口逆变换 × 玩家变换
Matrix portalTransform = exitPortal.transform * inverse(entryPortal.transform);
portalCamera.transform = portalTransform * playerCamera.transform;
```

---

## 玩家穿越传送门的瞬间

```
检测到玩家坐标穿过传送门平面
  → 把玩家坐标从入口映射到出口
  → 主摄像机切换到房间B
  → 世界没变，玩家位置变了
```

这就是为什么扔进传送门的箱子能从另一边飞出来——**箱子从没离开这个世界，只是坐标变了。**

---

## 递归传送门

站在传送门前能看到另一个传送门（套娃）：

```
需要从最深层往外逐层渲染：

Pass 1: 最深层视角 → RT_N
Pass 2: 上一层，传送门面贴 RT_N → RT_N-1
...
Pass N: 主场景，第一个传送门贴 RT_1 → Backbuffer
```

**这就是为什么 Portal 游戏限制递归深度**（通常 2-3 层）——深度每加一，多一个完整的 Pass。

---

## 难点：Oblique Near Clip Plane（斜近裁剪面）

```
问题：Portal Camera 的视锥会包含传送门平面后方的物体
     这些物体不应该被看见

解决：把 Portal Camera 的近裁剪面对齐到传送门所在平面
     传送门平面后方的所有物体被裁剪掉
```

---

## 同类技术

同一个原理（RT 贴到面上）还用在：

| 技术 | 说明 |
| :--- | :--- |
| **镜子** | 以镜面反射方向渲染一个摄像机 → RT 贴到镜面 |
| **监控屏幕** | 另一个位置的摄像机 → RT 贴到屏幕面 |
| **水面反射** | 以水面镜像摄像机渲染 → RT 贴到水面 |
| **反射探针** | 6 个方向渲染 Cubemap RT → 环境反射 |

---

## 相关概念

- [[Render-Target]] — Portal RT 的基础
- [[Deferred-Rendering]] — RT 和 Pass 的另一个重要应用
