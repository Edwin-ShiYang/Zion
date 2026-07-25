---
type: concept
created: 2026-07-23
last_updated: 2026-07-23
tags:
  - rendering
  - d3d11
  - shadow
aliases:
  - Shadow Map
  - Shadow Mapping
---

# Shadow Map

一句话直觉：**shadow map 是从 light 的视角渲染出来的一张 depth texture**。它不存颜色，只存“从 light 看过去，每个方向上最近的表面有多远”。

主渲染时，每个像素再去查这张 texture：如果当前像素离 light 更远，说明它被前面的东西挡住了，就在阴影里。

---

## 两个 Pass

```text
Shadow Pass / 阴影 pass
= camera is the light / 从 light 的视角渲染场景
= write depth only / 只写 depth，不写 color
= output is shadow map / 输出是一张 depth texture
```

```text
Main Pass / 主渲染 pass
= camera is the player camera / 从玩家 camera 渲染场景
= sample shadow map / 在 shader 里读取 shadow map
= compare current depth with stored light depth / 比较当前点和 shadow map 里的 depth
```

---

## Shadow Map 需要的 D3D11 对象

```text
m_shadowMapTexture = actual GPU texture / 真正存 shadow depth 的 GPU texture
m_shadowMapDSV     = Depth Stencil View / shadow pass 写 depth 用的 view
m_shadowMapSRV     = Shader Resource View / main pass shader 读取 depth 用的 view
```

注意：shadow map 通常**不需要** `RenderTargetView`，因为它不是写颜色的 render target。它是写 depth 的 texture。

```text
RTV = write color / 写颜色
DSV = write depth-stencil / 写 depth-stencil
SRV = shader reads resource / shader 读取资源
```

---

## Texture 本体为什么用 TYPELESS

Shadow map 的底层 texture 要被两种 view 使用：

```text
DSV = write depth / 写 depth
SRV = shader read depth / shader 读 depth
```

所以底层 resource 一般用：

```cpp
config.format = DXGI_FORMAT_R24G8_TYPELESS;
```

意思是：

```text
DXGI_FORMAT_R24G8_TYPELESS
= raw 24+8 bit storage / 只是 24+8 bit 的存储
= no fixed interpretation yet / 还没有固定解释成 depth、stencil 或 shader-readable value
```

如果 texture 本体直接写成：

```cpp
config.format = DXGI_FORMAT_D24_UNORM_S8_UINT;
```

它就被固定成 depth-stencil 格式。这样后面再创建 shader-readable 的 SRV 会很麻烦，很多情况下 D3D11 不允许用 depth-stencil format 直接当 shader texture 读。

所以正确直觉是：

```text
Texture resource uses TYPELESS / texture 本体不固定语义
View format decides interpretation / 每个 view 决定怎么解释同一块 memory
```

---

## DSV 为什么用 D24_UNORM_S8_UINT

```cpp
dsvDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
```

这是 shadow pass 写 depth 时使用的 view format。

```text
DXGI_FORMAT_D24_UNORM_S8_UINT
= D24_UNORM / 24-bit normalized depth，写 depth test 用的深度值
= S8_UINT / 8-bit unsigned stencil，整数 stencil 部分
```

也就是说：

```text
m_shadowMapTexture + m_shadowMapDSV
= use the typeless texture as a depth-stencil buffer / 把 typeless texture 当 depth-stencil buffer 写
```

写的时候不是 shader 在写 texture，而是 Output Merger / depth test 通过 DSV 写入 depth。

```cpp
m_deviceContext->OMSetRenderTargets(0, nullptr, m_shadowMapDSV);
```

---

## SRV 为什么用 R24_UNORM_X8_TYPELESS

```cpp
srvDesc.Format = DXGI_FORMAT_R24_UNORM_X8_TYPELESS;
```

这是 main pass 在 shader 里读取 shadow map 时使用的 view format。

```text
DXGI_FORMAT_R24_UNORM_X8_TYPELESS
= R24_UNORM / shader reads the 24-bit depth value
= X8_TYPELESS / ignore the 8-bit stencil part
```

也就是说：

```text
m_shadowMapTexture + m_shadowMapSRV
= use the same typeless texture as shader-readable depth / 把同一张 typeless texture 当 shader 可读的 depth texture
```

---

## 三个 Format 的关系

```text
Texture resource:
DXGI_FORMAT_R24G8_TYPELESS
= raw 24+8 bit storage / 未固定语义的 24+8 bit 存储

DSV:
DXGI_FORMAT_D24_UNORM_S8_UINT
= depth-stencil writing / shadow pass 写 depth-stencil

SRV:
DXGI_FORMAT_R24_UNORM_X8_TYPELESS
= shader-readable depth / main pass 读 depth，忽略 stencil
```

一句话：

```text
同一张 shadow map texture，shadow pass 通过 DSV 写 depth，main pass 通过 SRV 读 depth，所以底层 texture 用 TYPELESS，DSV/SRV 各自指定自己的解释格式。
```

---

## Texture Description

Shadow map 常见 descriptor：

```cpp
TextureDescriptionConfig config;
config.width = 2048;
config.height = 2048;
config.mipLevels = 1;
config.arraySize = 1;
config.format = DXGI_FORMAT_R24G8_TYPELESS;
config.sampleCount = 1;
config.sampleQuality = 0;
config.usage = D3D11_USAGE_DEFAULT;
config.bindFlags = D3D11_BIND_DEPTH_STENCIL | D3D11_BIND_SHADER_RESOURCE;
config.cpuAccessFlags = 0;
config.miscFlags = 0;
```

字段直觉：

```text
width / height       = shadow map resolution / shadow map 分辨率，常见 1024、2048、4096
mipLevels            = 1 / shadow map 通常不需要 mipmap
format               = R24G8_TYPELESS / 底层 texture 不固定解释方式
sampleCount          = 1 / shadow map 通常不开 MSAA
usage                = DEFAULT / GPU 写，GPU 读，CPU 不直接访问
bindFlags            = DEPTH_STENCIL | SHADER_RESOURCE / 既能创建 DSV，也能创建 SRV
cpuAccessFlags       = 0 / CPU 不直接访问
miscFlags            = 0 / 不生成 mipmap，不是 cubemap，不共享
```

Shadow map resolution 不一定要等于 window size。常见做法是固定正方形，例如 `2048 x 2048`，因为它表示 light 的阴影采样精度，不是屏幕大小。

---

## Shadow Map Sampler

Shadow map 第一版通常不要用 `SamplerMode::BILINEAR_WRAP`。

```text
BILINEAR_WRAP
= bilinear filtering + wrap address mode / 平滑采样 + UV 越界重复
```

Shadow map 的 UV 越界时不应该重复采样另一边的 depth，所以第一版通常用：

```cpp
// First version: avoid repeated depth values outside the shadow map
SamplerMode shadowSamplerMode = SamplerMode::POINT_CLAMP;
```

```text
POINT = read one depth texel directly / 直接读一个 depth texel
CLAMP = clamp UV outside 0..1 to the edge / UV 越界时夹到边缘，不重复平铺
```

后面要做 softer shadow 时，可以再考虑 `ComparisonSampler` 或 PCF，而不是直接用 ordinary `BILINEAR_WRAP`。

---

## 常见坑

- 底层 texture 用 `DXGI_FORMAT_D24_UNORM_S8_UINT`，然后又想创建 SRV：应改成 typeless resource + typed views。
- 创建了 RTV：普通 shadow map 写 depth，不写 color，所以需要 DSV，不需要 RTV。
- 忘了绑定 `D3D11_BIND_SHADER_RESOURCE`：后面无法创建 SRV。
- 忘了绑定 `D3D11_BIND_DEPTH_STENCIL`：后面无法创建 DSV。
- shadow pass 结束后还把 shadow map 作为 DSV 绑定，同时又在 pixel shader 里读它：同一资源不能同时写和读，main pass 前要解除冲突绑定。
- 对 shadow map 使用 `BILINEAR_WRAP`：UV 越界会重复采样另一侧 depth，第一版优先用 `POINT_CLAMP`。

---

## 相关链接

- [[03 - D3D11/D3D11.D3D11_TEXTURE2D_DESC]]
- [[03 - D3D11/D3D11.DepthBuffer]]
- [[03 - D3D11/D3D11.Texture]]
- [[rendering/Render-Target]]
