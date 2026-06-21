---
tags: [rendering, deferred-rendering, pass, gbuffer, d3d11]
aliases: [延迟渲染, Deferred, GBuffer, Pass, 渲染通道]
---

# Deferred Rendering（延迟渲染）

把"几何信息收集"和"光照计算"拆成两个 Pass，彻底解耦几何复杂度和光源数量。

---

## 核心概念：Pass

**Pass = 一次完整的"绑定 RT → Draw → 解绑"的过程。** 就像流水线上的一道工序。

```
Pass 三要素：
  输入  → 上一个 Pass 的 RT（当贴图读）
  处理  → 绑定 Shader，执行 Draw Call
  输出  → 写入某张 RT
```

### Pass 的两种形状

| 类型 | 做什么 | Draw Call |
| :--- | :--- | :--- |
| **几何 Pass** | 遍历场景所有 Mesh | 每个 Mesh 一次 |
| **全屏 Pass** | 铺一个全屏四边形，PS 处理每个像素 | 只有 1 次 |

---

## 为什么需要 Deferred？

### Forward Rendering 的问题

```
每个像素 = 遍历所有光源

100 个物体 × 50 个光源 = 5000 次光照计算
光源翻倍 → 开销翻倍

而且被遮挡的像素也算了，全部浪费
```

### Deferred 的解法

```
Forward:   物体数 × 光源数（含遮挡浪费）
Deferred:  只有屏幕可见像素 × 影响它的光源

被遮挡的像素在 Geometry Pass 就被 Depth Test 淘汰
根本不参与光照计算
```

---

## 完整 Pass 链

```
Pass 1: Shadow Pass
  摄像机：灯光视角
  输出：ShadowMap RT（深度纹理）

Pass 2: Geometry Pass（填充 GBuffer）
  输入：场景 Mesh + 材质贴图
  输出：GBuffer × 4 张 RT（MRT）
  ↓ GBuffer 内容固定，成为"快照"

Pass 3: SSAO（可选）
  输入：GBuffer_Depth + GBuffer_Normal
  输出：SSAO RT

Pass 4: Lighting Pass
  输入：所有 GBuffer + ShadowMap + SSAO
  处理：全屏四边形，PS 读 GBuffer 算 PBR 光照
  输出：HDR Scene RT

Pass 5: Bloom、Tonemap、UI...
  输出：Backbuffer → 屏幕
```

**数据单向流动：上一个 Pass 的 RT 是下一个 Pass 的输入，不会回头。**

---

## GBuffer 布局

```
GBuffer_0: R8G8B8A8   → [Albedo.RGB,   Roughness]
GBuffer_1: R16G16B16  → [Normal.XYZ,   Metallic]
GBuffer_2: R32_FLOAT  → [Depth]
GBuffer_3: R8G8B8A8   → [Emissive.RGB, AO]
```

---

## D3D11 完整实现

### 创建 GBuffer

```cpp
struct GBuffer {
    ID3D11Texture2D*          texture = nullptr;
    ID3D11RenderTargetView*   rtv     = nullptr;
    ID3D11ShaderResourceView* srv     = nullptr;
};

GBuffer gbuffer[4];

DXGI_FORMAT formats[] = {
    DXGI_FORMAT_R8G8B8A8_UNORM,
    DXGI_FORMAT_R16G16B16A16_FLOAT,
    DXGI_FORMAT_R32_FLOAT,
    DXGI_FORMAT_R8G8B8A8_UNORM,
};

for (int i = 0; i < 4; i++) {
    D3D11_TEXTURE2D_DESC desc = {};
    desc.Width     = width;
    desc.Height    = height;
    desc.Format    = formats[i];
    desc.BindFlags = D3D11_BIND_RENDER_TARGET | D3D11_BIND_SHADER_RESOURCE;
    desc.MipLevels = desc.ArraySize = desc.SampleDesc.Count = 1;

    device->CreateTexture2D(&desc, nullptr, &gbuffer[i].texture);
    device->CreateRenderTargetView(gbuffer[i].texture, nullptr, &gbuffer[i].rtv);
    device->CreateShaderResourceView(gbuffer[i].texture, nullptr, &gbuffer[i].srv);
}
```

### Geometry Pass

```cpp
void GeometryPass(ID3D11DeviceContext* ctx) {
    // 同时绑定 4 张 RT（MRT）
    ID3D11RenderTargetView* rtvs[] = {
        gbuffer[0].rtv, gbuffer[1].rtv, gbuffer[2].rtv, gbuffer[3].rtv
    };
    ctx->OMSetRenderTargets(4, rtvs, depthStencilView);

    float black[] = { 0, 0, 0, 0 };
    for (auto& rtv : rtvs) ctx->ClearRenderTargetView(rtv, black);
    ctx->ClearDepthStencilView(depthStencilView, D3D11_CLEAR_DEPTH, 1.0f, 0);

    ctx->VSSetShader(geometryVS, nullptr, 0);
    ctx->PSSetShader(geometryPS, nullptr, 0);

    for (Mesh& mesh : scene) {
        ctx->IASetVertexBuffers(0, 1, &mesh.vb, &stride, &offset);
        ctx->IASetIndexBuffer(mesh.ib, DXGI_FORMAT_R32_UINT, 0);
        ctx->PSSetShaderResources(0, 1, &mesh.albedoSRV);
        ctx->DrawIndexed(mesh.indexCount, 0, 0);
    }

    // ⚠️ 必须解绑——下一步要把这些纹理当 SRV 读
    ID3D11RenderTargetView* nullRTVs[4] = {};
    ctx->OMSetRenderTargets(4, nullRTVs, nullptr);
}
```

### Lighting Pass

```cpp
void LightingPass(ID3D11DeviceContext* ctx) {
    // 输出到 Backbuffer
    ctx->OMSetRenderTargets(1, &backbufferRTV, nullptr);

    // 把 GBuffer 绑定为输入纹理
    ID3D11ShaderResourceView* srvs[] = {
        gbuffer[0].srv, gbuffer[1].srv, gbuffer[2].srv, gbuffer[3].srv
    };
    ctx->PSSetShaderResources(0, 4, srvs);

    ctx->VSSetShader(fullscreenVS, nullptr, 0);
    ctx->PSSetShader(lightingPS, nullptr, 0);

    // 全屏四边形——触发每个像素跑一次 PS
    ctx->IASetVertexBuffers(0, 1, &fullscreenQuadVB, &stride, &offset);
    ctx->Draw(6, 0);

    ID3D11ShaderResourceView* nullSRVs[4] = {};
    ctx->PSSetShaderResources(0, 4, nullSRVs);
}
```

### Lighting Pass HLSL

```hlsl
Texture2D GBuf_Albedo   : register(t0);
Texture2D GBuf_Normal   : register(t1);
Texture2D GBuf_Depth    : register(t2);
Texture2D GBuf_Emissive : register(t3);

float4 PS_Lighting(float2 uv : TEXCOORD) : SV_Target0
{
    float4 albedoRough = GBuf_Albedo.Sample(s, uv);
    float4 normalMetal = GBuf_Normal.Sample(s, uv);
    float  depth       = GBuf_Depth.Sample(s, uv).r;
    float4 emissiveAO  = GBuf_Emissive.Sample(s, uv);

    float3 albedo    = albedoRough.rgb;
    float  roughness = albedoRough.a;
    float3 normal    = normalMetal.rgb * 2.0 - 1.0;
    float  metallic  = normalMetal.a;

    float3 finalColor = PBR(albedo, normal, depth, roughness, metallic);
    finalColor += emissiveAO.rgb;

    return float4(finalColor, 1.0);
}
```

### 主循环

```cpp
void RenderFrame() {
    GeometryPass(ctx);   // 场景 → GBuffer（4 张 RT）
    LightingPass(ctx);   // GBuffer → Backbuffer

    swapChain->Present(1, 0);
}
```

---

## Deferred 的代价

| 问题 | 原因 | 应对方案 |
| :--- | :--- | :--- |
| **带宽大** | GBuffer 每帧读写几十 MB | 压缩格式、减少通道数 |
| **透明物体难** | GBuffer 只存最近一层 | 透明物体单独用 Forward |
| **MSAA 很贵** | 多采样 × 多张 GBuffer | 用 TAA 代替 |
| **手机慎用** | 移动 GPU 不擅长大带宽 | Tile-Based Deferred |

---

## Forward vs Deferred

| 特性 | Forward | Deferred |
| :--- | :--- | :--- |
| **光源数量** | 受限（~16个） | 数百个 |
| **透明物体** | ✅ 天然支持 | ❌ 需要额外处理 |
| **MSAA** | ✅ 便宜 | ❌ 很贵 |
| **带宽** | 低 | 高 |
| **GBuffer 副产品** | ❌ 没有 | ✅ SSAO/SSR 免费 |
| **适合场景** | 移动端、透明物体多 | 3A 游戏、光源数量多 |

---

## 相关概念

- [[Render-Target]] — RT 和 MRT 的基础
- [[Portal-Rendering]] — RT 的另一种应用
