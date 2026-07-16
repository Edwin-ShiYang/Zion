---
type: concept
created: 2026-07-04
last_updated: 2026-07-04
tags: [math, physics, game-dev, movement, vector]
sources:
  - "[[raw/books/Game Physics Engine Development.pdf]]"
---

# Position / Velocity / Speed / Displacement

## 一句话直觉

基础 movement / physics integration 先记住三行：

```cpp
velocity = direction * speed;
displacement = velocity * deltaSeconds;
newPosition = oldPosition + displacement;
```

这三行回答三个问题：

1. **往哪、走多快？** → `velocity = direction * speed`
2. **这一帧实际走多少？** → `displacement = velocity * deltaSeconds`
3. **新位置在哪？** → `newPosition = oldPosition + displacement`

---

## 核心概念

| 名字 | 类型 | 意思 |
|---|---|---|
| `position` | `Vec3` | 物体在哪里 |
| `direction` | `Vec3` | 往哪个方向，通常长度为 1 |
| `speed` | `float` | 多快，只有大小 |
| `velocity` | `Vec3` | 方向 + 速度大小 |
| `deltaSeconds` | `float` | 当前这一帧经过了多少秒 |
| `displacement` | `Vec3` | 从旧位置到新位置的位移 |

---

## 三行公式

### 1. Direction + Speed → Velocity

```cpp
velocity = direction * speed;
```

`direction` 只表示“往哪”，`speed` 只表示“多快”。  
合起来之后，`velocity` 同时包含方向和速度大小。

### 2. Velocity + Time → Displacement

```cpp
displacement = velocity * deltaSeconds;
```

`velocity` 是每秒移动多少，乘以这一帧的时间，就得到这一帧实际移动多少。

### 3. Old Position + Displacement → New Position

```cpp
newPosition = oldPosition + displacement;
```

位置本身不会直接“变成速度”。位置是被位移更新的。

---

## 常见合并写法

学习时优先记三行，不要一开始就合并。  
但代码里常会写成：

```cpp
newPosition = oldPosition + direction * speed * deltaSeconds;
```

这只是三行公式的压缩版。

---

## 反推速度

如果已知两个位置和经过的时间，可以反推平均速度：

```cpp
displacement = newPosition - oldPosition;
velocity = displacement / deltaSeconds;
```

这常用于：

- root motion 反推速度
- replay / recorded motion
- 根据上一帧和当前帧位置估算物体速度

---

## 和 acceleration / gravity 的关系

之后加入 acceleration 时，只是在前面多一步更新 velocity：

```cpp
velocity += acceleration * deltaSeconds;
displacement = velocity * deltaSeconds;
newPosition = oldPosition + displacement;
```

如果 acceleration 是重力：

```cpp
velocity += gravityAcceleration * deltaSeconds;
displacement = velocity * deltaSeconds;
newPosition = oldPosition + displacement;
```

---

## 常见坑

### 忘记乘 `deltaSeconds`

错误：

```cpp
position += velocity;
```

这会让移动速度和 FPS 绑定。

正确：

```cpp
position += velocity * deltaSeconds;
```

### 混淆 `speed` 和 `velocity`

```cpp
float speed;   // 只有大小
Vec3 velocity; // 方向 + 大小
```

### 混淆 `direction` 和 `displacement`

```cpp
Vec3 direction;    // 长度通常是 1
Vec3 displacement; // 有方向，也有距离
```

---

## 相关链接

- [[04 - Math/MP.MathUtils.DotProduct]]
- [[04 - Math/MP.MathUtils.CrossProduct]]
- [[04 - Math/Collision.Z-Aligned-Capsule-Overlap]]

