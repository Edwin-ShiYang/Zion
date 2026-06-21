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

```hlsl
float3 idealReflectionDir = normalize( pixelToCameraDir + pixelToLightDir ); // "H" 半角向量
float specularDot = saturate( dot( idealReflectionDir, pixelNormalWorldSpace ) );
float specularStrength = glossiness * lightColor.a * pow( specularDot, specularExponent );
```

- `H`（半角向量）= 视线方向和光线方向的中间方向
- `H` 和法线越接近，高光越亮（镜面反射条件）
- `pow(specularDot, specularExponent)` 让高光集中成一个小亮斑：
  - `specularExponent` 越大 → 高光越小越锐利（越光滑的表面）
  - `specularExponent` 越小 → 高光越大越柔和（越粗糙的表面）

```hlsl
float specularExponent = RangeMap( glossiness, 0.f, 1.f, 1.f, 32.f );
```

---

## Emissive：自发光

```hlsl
float3 emissiveLight = diffuseTexel.rgb * emissiveness;
```

不依赖任何光源，直接由贴图的自发光通道决定亮度，常用于灯泡、霓虹灯、屏幕等。

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
