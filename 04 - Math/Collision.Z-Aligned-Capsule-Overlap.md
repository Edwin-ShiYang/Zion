---
type: concept
created: 2026-06-28
last_updated: 2026-06-28
tags: [collision, capsule, character-controller, math, game-dev]
sources: []
---

# Z-Aligned Capsule Overlap

> 用于 third-person character / Unreal-style character capsule 的重叠检测。

---

## 核心结论

对于角色用的 capsule，通常可以假设它是 **Z-aligned / upright capsule**：

```text
capsule axis parallel to world Z
```

这和 Unreal `ACharacter` 的 capsule 思路接近：角色身体碰撞通常保持竖直，不跟随动画骨骼乱转。

---

## 参数语义

如果采用 Unreal-style 语义：

```cpp
center      = capsule center
radius      = capsule radius
halfHeight  = center 到 capsule 顶部/底部的距离
```

那么：

```text
totalHeight = halfHeight * 2
```

注意：`halfHeight` 不是 capsule 内部 bone/segment 的半长。

---

## Bone Start / Bone End

Capsule 可以理解成：

```text
line segment + radius
```

这里的 line segment 是上下两个半球球心之间的线段。

如果：

```cpp
halfHeight = center 到 capsule 顶/底的距离
```

那么 bone 的半长是：

```cpp
boneHalfHeight = halfHeight - radius;
```

所以：

```cpp
Vec3 boneStart = Vec3(center.x, center.y, center.z - halfHeight + radius);
Vec3 boneEnd   = Vec3(center.x, center.y, center.z + halfHeight - radius);
```

或者写得更清楚：

```cpp
float boneHalfHeight = halfHeight - radius;

Vec3 boneStart = Vec3(center.x, center.y, center.z - boneHalfHeight);
Vec3 boneEnd   = Vec3(center.x, center.y, center.z + boneHalfHeight);
```

需要保证：

```cpp
halfHeight >= radius
```

否则 capsule 会退化或参数非法。

---

## Overlap 思路

两个 Z-aligned capsules 的 overlap 可以分三种情况：

### 1. A 在 B 下方

```cpp
boneEndA.z <= boneStartB.z
```

最近点是：

```text
boneEndA 和 boneStartB
```

### 2. B 在 A 下方

```cpp
boneEndB.z <= boneStartA.z
```

最近点是：

```text
boneEndB 和 boneStartA
```

### 3. Bone 的 Z 区间重叠

这时最近距离是水平距离：

```text
distance between centerA.xy and centerB.xy
```

---

## Kissing 是否算 Overlap

本项目约定：

```text
kissing / just touching 不算 overlap
```

所以判断条件是：

```cpp
distanceSq < (radiusA + radiusB)^2
```

如果：

```cpp
distanceSq >= (radiusA + radiusB)^2
```

则不算 overlap。

---

## Reference Implementation

```cpp
//-----------------------------------------------------------------------------------------------
bool DoZAlignedCapsulesOverlap3D(
    Vec3 centerA,
    float radiusA,
    float halfHeightA,
    Vec3 centerB,
    float radiusB,
    float halfHeightB
)
{
    Vec3 boneStartA = Vec3( centerA.x, centerA.y, centerA.z - halfHeightA + radiusA );
    Vec3 boneEndA   = Vec3( centerA.x, centerA.y, centerA.z + halfHeightA - radiusA );

    Vec3 boneStartB = Vec3( centerB.x, centerB.y, centerB.z - halfHeightB + radiusB );
    Vec3 boneEndB   = Vec3( centerB.x, centerB.y, centerB.z + halfHeightB - radiusB );

    float radiiSumSquared = ( radiusA + radiusB ) * ( radiusA + radiusB );

    if ( boneEndA.z <= boneStartB.z )
    {
        if ( GetDistanceSquared3D( boneEndA, boneStartB ) >= radiiSumSquared )
        {
            return false;
        }
    }
    else if ( boneStartA.z >= boneEndB.z )
    {
        if ( GetDistanceSquared3D( boneStartA, boneEndB ) >= radiiSumSquared )
        {
            return false;
        }
    }
    else if ( GetDistanceSquared2D( Vec2( centerA.x, centerA.y ), Vec2( centerB.x, centerB.y ) ) >= radiiSumSquared )
    {
        return false;
    }

    return true;
}
```

---

## 更直接的写法

同样逻辑也可以写成直接 `return`：

```cpp
bool DoZAlignedCapsulesOverlap3D(
    Vec3 centerA,
    float radiusA,
    float halfHeightA,
    Vec3 centerB,
    float radiusB,
    float halfHeightB
)
{
    float radiiSum = radiusA + radiusB;
    float radiiSumSquared = radiiSum * radiiSum;

    float boneHalfHeightA = halfHeightA - radiusA;
    float boneHalfHeightB = halfHeightB - radiusB;

    Vec3 boneStartA = Vec3( centerA.x, centerA.y, centerA.z - boneHalfHeightA );
    Vec3 boneEndA   = Vec3( centerA.x, centerA.y, centerA.z + boneHalfHeightA );

    Vec3 boneStartB = Vec3( centerB.x, centerB.y, centerB.z - boneHalfHeightB );
    Vec3 boneEndB   = Vec3( centerB.x, centerB.y, centerB.z + boneHalfHeightB );

    if ( boneEndA.z <= boneStartB.z )
    {
        return GetDistanceSquared3D( boneEndA, boneStartB ) < radiiSumSquared;
    }

    if ( boneStartA.z >= boneEndB.z )
    {
        return GetDistanceSquared3D( boneStartA, boneEndB ) < radiiSumSquared;
    }

    return GetDistanceSquared2D(
        Vec2( centerA.x, centerA.y ),
        Vec2( centerB.x, centerB.y )
    ) < radiiSumSquared;
}
```

---

## 常见坑

### 1. 混淆 `halfHeight` 和 `boneHalfHeight`

Unreal-style：

```text
halfHeight = center 到 capsule 顶/底的距离
```

Bone segment：

```text
boneHalfHeight = halfHeight - radius
```

如果直接用：

```cpp
center.z ± halfHeight
```

作为 boneStart / boneEnd，就会把 capsule 变得比真实更长。

### 2. 中间 case 不要用 3D center distance

当两个 bone 的 Z 区间重叠时，最近距离是 XY 距离，而不是 center 到 center 的 3D 距离：

```cpp
GetDistanceSquared2D(centerA.xy, centerB.xy)
```

### 3. 函数名要表达限制

`DoZAlignedCapsulesOverlap3D` 比 `DoCapsulesOverlap3D` 更准确，因为这个版本只支持竖直 capsule。

任意方向 capsule 应使用：

```cpp
startA, endA, radiusA
startB, endB, radiusB
```

并计算两条 segment 的最近距离。

---

## Related

- [[04 - Math]]
