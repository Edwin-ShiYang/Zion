---
tags: [rendering, lighting, blinn-phong, hlsl, d3d11]
aliases: [Blinn-Phong, 光照模型, Diffuse, Specular, Emissive]
---

# Blinn-Phong 光照模型

经典的实时光照近似模型，把表面接收到的光拆成三部分分别计算，再加起来：

```
最终颜色 = Diffuse（漫反射） + Specular（高光） + Emissive（自发光）
```

---

## 三种光的来源

| 类型 | 是什么 | 例子 |
| :--- | :--- | :--- |
| **Diffuse**（漫反射） | 光照到表面后均匀散开 | 白天看到的物体本色 |
| **Specular**（高光） | 光集中反射进眼睛的亮斑 | 金属/塑料表面的亮点 |
| **Emissive**（自发光） | 物体自己发光，不需要外部光源 | 灯泡、屏幕、发光贴图 |

---

## 核心：点乘 = 夹角余弦

两个**单位向量**点乘，结果是它们夹角的 cos 值：

```
dot(A, B) = cos(夹角)

夹角 = 0°   → dot = 1.0   完全同向
夹角 = 90°  → dot = 0.0   垂直
夹角 = 180° → dot = -1.0  完全反向
```

---

## Diffuse：Lambert's Law

```hlsl
float sunlightStrength = c_sunColor.a *
    saturate( RangeMap( dot( -c_sunNormal, pixelNormalWorldSpace ), -sunAmbience, 1.0, 0.0, 1.0 ) );

float3 diffuseLightFromSun = sunlightStrength * c_sunColor.rgb;
```

```
-c_sunNormal          = 光线方向反过来（指向光源）
pixelNormalWorldSpace = 这个像素表面的法线方向
```

```
        光源方向 ↖
                  \
                   \  ↑ 表面法线
                    \ |
              ───────●─────── 表面
```

- 光线**正对**表面（法线与"指向光源"方向重合）→ dot = 1 → **受光最强**
- 光线**斜着照**表面 → dot 在 0~1 之间 → **受光减弱**
- 表面**背对光源** → dot < 0 → **完全没光**（被 `saturate` 截断为 0）

物理直觉：正对太阳站立晒得最热，侧身站没那么热，背对完全晒不到。

> `RangeMap` 在这里把 `[-sunAmbience, 1.0]` 重新映射到 `[0.0, 1.0]`，
> `sunAmbience` 控制背光面是否还留一点点环境光（通常设 0，不想要全局最低环境光）。

### RangeMap 在这里做什么

```hlsl
float RangeMap( float inValue, float inStart, float inEnd, float outStart, float outEnd )
{
    float fraction = (inValue - inStart) / (inEnd - inStart);
    float outValue = outStart + fraction * (outEnd - outStart);
    return outValue;
}
```

点乘结果范围是 `[-1, 1]`，但颜色强度需要 `[0, 1]`，直接用 dot 的值背光面会是负数颜色，没有意义，所以要重映射。

### sunAmbience 控制映射的起点

```hlsl
RangeMap( dot, -sunAmbience, 1.0, 0.0, 1.0 )
```

| sunAmbience | dot = -1（完全背光） | dot = 0（侧面） | 效果 |
| :--- | :--- | :--- | :--- |
| `0` | 0（纯黑，截断） | 0（刚好黑） | 背光面纯黑，对比强，边界硬 |
| `0.3` | 0（截断） | ≈0.23（比之前亮） | 背光面有微弱亮度，过渡柔和 |

```
sunAmbience = 0：  ☀️受光渐变 → 🌑 背光突然全黑，立体感强但边界硬
sunAmbience = 0.3：☀️受光渐变 → 🌗 背光也有微弱亮度，过渡更柔和、更"平"
```

本质是在模拟"间接光/环境光"——现实中背光面也会被周围反射的光照亮一点，不会是纯黑。`sunAmbience` 越大，物体越"平"立体感越弱；越小，对比越强立体感越强但更"硬"。

---

## Specular：Blinn-Phong 高光

高光回答的问题和 Diffuse 不一样：**这个反射的光，是不是正好弹进了你的眼睛？**

```
拿一个光滑的球对着灯光转动，会看到一个亮斑随视角移动
亮斑出现的位置 = 反射光线恰好对准眼睛的地方
```

### 半角向量 H（Blinn 的近似技巧）

正统 Phong 要先算"反射光线"再跟视线比较，比较麻烦。Blinn-Phong 用一个近似：

```hlsl
float3 idealReflectionDir = normalize( pixelToCameraDir + pixelToLightDir ); // "H" 半角向量
```

```
pixelToCameraDir = 表面 → 摄像机 方向（视线方向 V）
pixelToLightDir  = 表面 → 光源 方向（光线方向 L）
把 V 和 L 加起来再归一化 → 得到正中间方向 H
```

```
        L（光源方向）
         \
          \   H（V 和 L 的中间方向）
           \ ↗
    ────────●──────── 表面
           ↗
          /
         V（视线方向）
```

**核心近似：H 越接近法线方向，就越接近"反射光恰好弹进眼睛"的条件。**

### 用点乘判断 H 和法线有多接近

```hlsl
float specularDot = saturate( dot( idealReflectionDir, pixelNormalWorldSpace ) );
```

逻辑和 Diffuse 一样：H 和法线完全重合 → dot = 1 → 高光最亮；夹角变大 → dot 变小 → 高光变暗。

### 用 pow 把高光"收紧"成一个点

只用 `specularDot` 高光会是一大片模糊亮区，不像真实高光斑点：

```hlsl
float specularStrength = glossiness * lightColor.a * pow( specularDot, specularExponent );
```

```
specularDot = 0.9, specularExponent = 1   →  0.9^1  = 0.9    （还很亮）
specularDot = 0.9, specularExponent = 32  →  0.9^32 = 0.034  （已经很暗）
```

指数越高，dot 稍微偏离 1 亮度就暴跌——高光只集中在 H 几乎完美对准法线的一小块区域，形成锐利亮点。

### specularExponent 由光滑度（glossiness）决定

```hlsl
float specularExponent = RangeMap( glossiness, 0.f, 1.f, 1.f, 32.f );
```

| glossiness | specularExponent | 视觉效果 |
| :--- | :--- | :--- |
| 0（粗糙，如石头） | 1 | 高光又大又暗，几乎看不出来 |
| 1（光滑，如镜子） | 32 | 高光又小又亮，像镭射点 |

这就是为什么调材质时"光滑度"滑条对视觉效果影响特别明显：镜子/抛光金属高光是尖锐小亮点，木头/石头高光是一片模糊亮区，几乎融入漫反射。

---

## Emissive：自发光

```hlsl
float3 emissiveLight = diffuseTexel.rgb * emissiveness;
```

```
diffuseTexel  = 贴图本身的颜色
emissiveness  = 自发光强度（通常从 SpecGlossEmit 贴图的蓝色通道读出来）
```

不依赖任何光源，直接由贴图的自发光通道决定亮度：`emissiveness = 0` 不发光（正常受光照影响），`emissiveness = 1` 自己发光（哪怕全黑环境也亮）。常用于灯泡、霓虹灯字、屏幕、发光符文等。

---

## 最终合成公式

```hlsl
float3 finalRGB = (saturate(totalDiffuseLight) * diffuseColor.rgb)
                 + (totalSpecularLight * specularity)
                 + emissiveLight;
```

| 项 | 含义 |
| :--- | :--- |
| `saturate(totalDiffuseLight) * diffuseColor.rgb` | 漫反射强度 × 物体本身颜色 |
| `totalSpecularLight * specularity` | 高光强度 × 表面反光程度（金属度/specularity 贴图控制） |
| `emissiveLight` | 自发光，直接加上，不受光照影响 |

### 为什么漫反射要乘物体颜色，高光不用

```
漫反射：光被物体"吸收再散射"，带上物体本身颜色
        红色苹果在白光下反射红光，因为表面吸收了其他颜色

高光：  光在表面直接镜面反射，大多数材质的高光接近光源颜色（白色）
        几乎不受物体本身颜色影响
        （红苹果上的高光斑点看起来是白色的，不是红色的）
```

---

## 点光源 / 聚光灯：衰减与聚光锥

太阳光无限远、方向永远一样、强度不随距离变化。点光源/聚光灯不一样：有具体位置，离得越远越暗，超出范围完全没光；聚光灯还多一层"只朝一个方向照"的限制。

### 衰减半径（Falloff）

```hlsl
float distToLight = length( pixelToLightDisp );
float falloff = saturate( RangeMap( distToLight, innerRadius, outerRadius, 1.f, 0.f ) );
falloff = SmoothStep3( falloff );
```

```
innerRadius 内：满强度（falloff = 1）
outerRadius 外：完全没光（falloff = 0）
中间：渐变
```

```
        outerRadius
       ╱─────────╲
      │ innerRadius │
      │      ●      │   ← 光源在中心
       ╲─────────╱
```

用 `SmoothStep3`（`3x² - 2x³`）代替直接线性插值，让两端变化慢、中间变化快，过渡更自然，肉眼看不出突变边界。

### 聚光锥（Penumbra）

点光源四面八方发光，聚光灯像手电筒只照一个锥形范围。判断"在不在锥形里"还是靠点乘：

```hlsl
float penumbra = saturate( RangeMap( dot( c_spotForward, lightToPixelDir ), outerPenumbraDot, innerPenumbraDot, 0.f, 1.f ) );
penumbra = SmoothStep3( penumbra );
```

```
c_spotForward   = 聚光灯朝向的方向
lightToPixelDir = 光源 → 像素 的方向

dot 接近 1  →  像素在聚光灯正前方中心 → 最亮
dot 接近 0  →  像素在锥形边缘 → 变暗
dot 更小    →  完全在锥形外 → 没光
```

```
                聚光灯
                  ●
                 ╱│╲
        外锥边界 ╱ │ ╲ 外锥边界
              ╱ 内锥 │   ╲
            ╱─────┼─────╲
       完全照亮   过渡区   完全照亮
```

`innerPenumbraDot` / `outerPenumbraDot` 是**提前转换成 cos 值**存起来的角度阈值：

```cpp
// C++ 端，内锥 20°，外锥 30°
light.c_innerDotThreshold = cos( DegreesToRadians( 20.f ) );
light.c_outerDotThreshold = cos( DegreesToRadians( 30.f ) );
```

提前转 cos 是因为 GPU 的 `dot()` 算出来的就是 cos 值，两边统一比较 cos，避免在 Shader 里反复算开销很大的 `acos`。

### 一套公式同时处理点光源和聚光灯

```hlsl
innerPenumbraDot = -1; // 用于 point light
outerPenumbraDot = -2; // 用于 point light
```

任何方向的 `dot()` 结果都 ≥ -1，所以 `RangeMap(dot, -2, -1, 0, 1)` 这种范围永远映射出 1（全亮）——点光源借用同一套聚光灯公式，靠"开区间足够大"实现"四面八方都满强度"，不用写两套逻辑。

---

## 整体结构（伪代码）

```hlsl
float3 totalDiffuseLight  = 0;
float3 totalSpecularLight = 0;

// 1. 太阳光（平行光，方向固定，无衰减）
totalDiffuseLight  += 太阳的漫反射;
totalSpecularLight += 太阳的高光;

// 2. 点光源 / 聚光灯（有位置、有衰减半径）
for each light:
    totalDiffuseLight  += 这个光的漫反射;
    totalSpecularLight += 这个光的高光;

// 3. 自发光
float3 emissiveLight = diffuseTexel.rgb * emissiveness;

// 4. 合成
finalColor.rgb = saturate(totalDiffuseLight) * diffuseColor.rgb
               + totalSpecularLight * specularity
               + emissiveLight;
```

---

## 为什么光照计算在 World Space

Vertex Shader 把顶点变换到 Clip Space 用于渲染（裁剪、透视除法、画在哪个像素），
但同时把 **World Space** 的位置/法线/切线另外传给 Pixel Shader：

```hlsl
output.v_position    = clipPos;     // 给 GPU 用来确定画在哪
output.v_worldPos    = worldPos.xyz;   // 给 Pixel Shader 算光照用
output.v_worldNormal = worldNormal.xyz;
```

因为光源位置、太阳方向都是世界坐标，光照计算必须在同一个坐标系里做点乘/距离计算：

```hlsl
float3 pixelToLightDisp = lightPos - input.v_worldPos; // 都在 World Space
```

一个顶点的位置被算了两次用途：留一份在 World Space 做光照，另一份继续往下转换到 Clip Space 用于渲染。

---

## 相关概念

- [[03 - D3D11/D3D11.ConstantBuffer]] — Light/Camera/Model 等数据如何传进 Shader
- [[Render-Target]] — Pixel Shader 输出最终写入哪里
