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

## D3D11 创建 RT

```cpp
D3D11_TEXTURE2D_DESC desc = {};
desc.Width     = 1920;
desc.Height    = 1080;
desc.Format    = DXGI_FORMAT_R8G8B8A8_UNORM;
desc.BindFlags = D3D11_BIND_RENDER_TARGET       // 可写
               | D3D11_BIND_SHADER_RESOURCE;    // 可读
desc.MipLevels = 1;
desc.ArraySize = 1;
desc.SampleDesc.Count = 1;

ID3D11Texture2D* tex = nullptr;
device->CreateTexture2D(&desc, nullptr, &tex);

// 两个 View，对应两种用途
ID3D11RenderTargetView*   rtv = nullptr;  // 写
ID3D11ShaderResourceView* srv = nullptr;  // 读
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
// 一次 Draw Call，PS 同时写入 4 张纹理
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
