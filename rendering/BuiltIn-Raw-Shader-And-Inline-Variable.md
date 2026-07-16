---
type: concept
created: 2026-07-14
last_updated: 2026-07-14
tags: [rendering, shader, cplusplus, engine, post-process]
sources: []
---

# Built-in Raw Shader 和 inline 变量

## 一句话直觉

如果一个 shader 是 engine 通用功能，例如 Tone Mapping、Bright Pass、Fullscreen Copy，可以先写成 engine 内置的 raw shader source。  
如果 raw shader source 放在 header 里，变量建议用 `inline`，这样多个 `.cpp` include 不会链接报重复定义。

---

## 为什么 Tone Mapping 可以是 engine built-in shader

Tone Mapping 不是某个 game 的特殊逻辑，而是 renderer 的通用 post-process：

```txt
SceneColorHDR → Tone Mapping → Back Buffer
```

所以它更适合属于 engine：

```txt
Engine Renderer / PostProcess / ToneMapping
```

Game 只负责调参数：

```cpp
exposure
bloomThreshold
bloomIntensity
```

而不是每个 game 都重新写一份 Tone Mapping shader。

---

## raw shader source 的适用场景

适合写成 raw string 的 shader：

- Tone Mapping
- Bright Pass
- Fullscreen Copy
- Error / fallback shader
- Debug simple shader

不太适合写成 raw string 的 shader：

- 经常被美术或技术美术调的复杂 VFX shader
- 大型 PBR shader
- 需要频繁 hot reload 的材质 shader

这些更适合放 `.hlsl` 文件。

---

## header 里为什么要用 inline

如果在 `.hpp` 里直接写：

```cpp
char const* g_toneMappingShaderSource = R"(
    // hlsl source
)";
```

然后多个 `.cpp` include 这个 header：

```cpp
Renderer.cpp
Game.cpp
PostProcess.cpp
```

每个 `.cpp` 都会生成一个同名全局变量定义。链接时可能报：

```txt
multiple definition
```

加上 `inline`：

```cpp
inline char const* g_toneMappingShaderSource = R"(
    // hlsl source
)";
```

意思是：

```txt
这个变量定义可以出现在多个 translation unit 里，linker 会把它们当成同一个实体。
```

简单记法：

> header 里的全局变量如果要被多个 `.cpp` include，通常要用 `inline`。

---

## 推荐写法

```cpp
#pragma once

namespace BuiltInShaders
{
inline constexpr char const* ToneMapping = R"(
struct VertexInput
{
    float3 a_position    : VERTEX_POSITION;
    float4 a_color       : VERTEX_COLOR;
    float2 a_uvTexCoords : VERTEX_UVTEXCOORDS;
};

struct VertexToPixel
{
    float4 position : SV_Position;
    float2 uv       : TEXCOORD0;
};

Texture2D<float4> t_hdrTexture : register(t5);
SamplerState      s_hdrSampler : register(s5);

VertexToPixel VertexMain(VertexInput input)
{
    VertexToPixel output;
    output.position = float4(input.a_position, 1.0f);
    output.uv = float2(input.a_uvTexCoords.x, 1.0f - input.a_uvTexCoords.y);
    return output;
}

float3 ToneMapReinhard(float3 hdrColor)
{
    return hdrColor / (hdrColor + 1.0f);
}

float3 LinearToGamma(float3 linearColor)
{
    return pow(saturate(linearColor), 1.0f / 2.2f);
}

float4 PixelMain(VertexToPixel input) : SV_Target0
{
    float3 hdrColor = t_hdrTexture.Sample(s_hdrSampler, input.uv).rgb;
    float exposure = 2.5f;

    float3 mappedColor = ToneMapReinhard(hdrColor * exposure);
    float3 displayColor = LinearToGamma(mappedColor);

    return float4(displayColor, 1.0f);
}
)";
}
```

---

## 更完整的做法

以后可以让 `Renderer` 支持：

```cpp
Shader* CreateShaderFromSource(
    char const* shaderName,
    char const* shaderSource,
    VertexType vertexType
);
```

然后 engine 初始化时创建：

```cpp
m_toneMappingShader = CreateShaderFromSource(
    "BuiltIn_ToneMapping",
    BuiltInShaders::ToneMapping,
    VertexType::VERTEX_PCUTBN
);
```

---

## 相关链接

- [[rendering/Render-Target]]
- [[rendering/Deferred-Rendering]]
- [[rendering/Bloom-HDR-PostProcess-Pipeline]]

