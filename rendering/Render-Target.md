---
tags: [rendering, render-target, mrt, d3d11]
aliases: [RT, Render Target, 渲染目标, MRT, Multiple Render Targets]
---

# Render Target（渲染目标）

GPU 渲染时"画到哪里"的目标缓冲区。就像画画时指定"画在哪张纸上"——默认画在屏幕（Backbuffer），但也可以先画到一张"中间纸"上，再用这张纸做进一步处理。

---

## RT 与 Texture 的关系

Render Target **本质就是一张 Texture**，只是它有双重身份：

```
同一块 GPU 内存

  作为 RTV（Render Target View）→ 写（当画布）
  作为 SRV（Shader Resource View）→ 读（当贴图）

⚠️ 不能同时读写——必须先解绑 RTV，再绑定 SRV
```

---

## Backbuffer vs Off-screen RT

| 类型 | 说明 | 用途 |
| :--- | :--- | :--- |
| **Backbuffer** | 交换链的一部分，最终显示到屏幕 | 最终输出 |
| **Off-screen RT** | 自定义的中间缓冲 | 后处理、阴影、反射 |
| **GBuffer**（多张 RT）| 延迟渲染的多张 RT | Deferred Rendering |
| **Shadow Map** | 以灯光视角渲染深度 | 阴影 |
| **Cubemap RT** | 6 面 RT | 环境映射、反射探针 |

---

## 像素格式选择

| 格式 | 用途 | 精度 |
| :--- | :--- | :--- |
| `R8G8B8A8_UNORM` | 普通颜色（0-1） | 8 bit/通道 |
| `R16G16B16A16_FLOAT` | HDR、法线 | 16 bit float |
| `R32_FLOAT` | 深度、精确数据 | 32 bit float |
| `R11G11B10_FLOAT` | HDR 无 Alpha | 11/10 bit |
| `D24_UNORM_S8_UINT` | Depth-Stencil | 24+8 bit |

---

## `DXGI_FORMAT_R16G16B16A16_FLOAT`

这个格式表示一张 RGBA texture，每个像素有 4 个通道：

```text
R = Red
G = Green
B = Blue
A = Alpha
```

每个通道是 16-bit float：

```text
R16 + G16 + B16 + A16 = 64 bits = 8 bytes / pixel
```

它和 `UNORM` 格式最大的区别是：**FLOAT 不会被限制在 `0.0 ~ 1.0`**。

```text
UNORM = unsigned normalized integer，通常映射到 0.0 ~ 1.0
FLOAT = floating point number，可以表示超过 1.0 的值
```

例如普通颜色格式：

```cpp
desc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
```

它每个通道是 8-bit normalized integer：

```text
0   -> 0.0
255 -> 1.0
```

如果光照计算得到 `4.0`，写入 `UNORM` render target 时会被夹到 `1.0`，亮度细节丢失。

而 HDR render target 常用：

```cpp
desc.Format = DXGI_FORMAT_R16G16B16A16_FLOAT;
```

它可以先保存超过 1 的光照结果：

```text
0.0
0.5
1.0
2.0
4.0
10.0
```

后面再通过 tone mapping 把 HDR 值压回屏幕能显示的 `0.0 ~ 1.0`。

常见用途：

- HDR lighting buffer
- Bloom 前的高亮颜色
- Deferred Rendering 的 GBuffer 中间数据
- 需要负数或超过 `1.0` 的中间计算结果

---

## D3D11 创建 RT

```cpp
D3D11_TEXTURE2D_DESC desc = {};
desc.Width     = 1920;
desc.Height    = 1080;
desc.Format    = DXGI_FORMAT_R8G8B8A8_UNORM;
desc.BindFlags = D3D11_BIND_RENDER_TARGET       // Writable as a render target
               | D3D11_BIND_SHADER_RESOURCE;    // Readable as a shader resource
desc.MipLevels = 1;
desc.ArraySize = 1;
desc.SampleDesc.Count = 1;

ID3D11Texture2D* tex = nullptr;
device->CreateTexture2D(&desc, nullptr, &tex);

// Two views for two different uses
ID3D11RenderTargetView*   rtv = nullptr;  // Write
ID3D11ShaderResourceView* srv = nullptr;  // Read
device->CreateRenderTargetView(tex, nullptr, &rtv);
device->CreateShaderResourceView(tex, nullptr, &srv);
```

---

## Multiple Render Targets（MRT）

### 为什么需要 MRT？

[[Deferred-Rendering|延迟渲染]] 的 GBuffer 需要同时存储多种信息：

```
一个像素的 PBR 光照需要：
  Albedo (RGB) + Normal (XYZ) + Depth + Metallic + Roughness + ...
  
一张 RT 只有 4 个通道（RGBA），根本装不下
→ 需要多张 RT 同时接收输出
```

### 没有 MRT 的代价

```
❌ 不用 MRT：场景渲染 4 遍（顶点变换 × 4）
✅ 用 MRT：  场景渲染 1 遍，PS 同时输出到 4 张 RT
```

### D3D11 绑定多张 RT

```cpp
ID3D11RenderTargetView* rtvs[] = {
    rtv_albedo,    // slot 0 → SV_Target0
    rtv_normal,    // slot 1 → SV_Target1
    rtv_depth,     // slot 2 → SV_Target2
    rtv_emissive,  // slot 3 → SV_Target3
};
context->OMSetRenderTargets(4, rtvs, depthStencilView);
// One draw call; the pixel shader writes to four textures at once
```

### HLSL 对应写法

```hlsl
struct PS_OUTPUT
{
    float4 Albedo   : SV_Target0;
    float4 Normal   : SV_Target1;
    float  Depth    : SV_Target2;
    float4 Emissive : SV_Target3;
};

PS_OUTPUT PS_GBuffer(VS_OUTPUT input)
{
    PS_OUTPUT o;
    o.Albedo   = float4(albedoTex.Sample(s, input.uv).rgb, roughness);
    o.Normal   = float4(input.normal * 0.5 + 0.5, metallic);
    o.Depth    = input.posH.z / input.posH.w;
    o.Emissive = float4(emissive, ao);
    return o;
}
```

---

## 相关概念

- [[Deferred-Rendering]] — MRT 最主要的使用场景
- [[Portal-Rendering]] — RT 的另一个经典应用
