---
title: WebGLRenderingContext.useProgram()
slug: Web/API/WebGLRenderingContext/useProgram
translation_of: Web/API/WebGLRenderingContext/useProgram
---
{{APIRef("WebGL")}}

[WebGL API](/ja/docs/Web/API/WebGL_API) の **`WebGLRenderingContext.useProgram()`** メソッドは、指定した {{domxref("WebGLProgram")}} を現在の描画ステートの一部として設定します。

## 構文

```
void gl.useProgram(program);
```

### 引数

- program
  - : 使用する {{domxref("WebGLProgram")}}。

### 返り値

ありません。

## 例

```js
var program = gl.createProgram();

// Attach pre-existing shaders
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);

gl.linkProgram(program);
gl.useProgram(program);
```

## 仕様策定状況

| 仕様                                                                                     | 策定状況                             | コメント                        |
| ---------------------------------------------------------------------------------------- | ------------------------------------ | ------------------------------- |
| {{SpecName('WebGL', "#5.14.9", "useProgram")}}                             | {{Spec2('WebGL')}}             | 初回定義。                      |
| {{SpecName('OpenGL ES 2.0', "glUseProgram.xml", "glUseProgram")}} | {{Spec2('OpenGL ES 2.0')}} | OpenGL API のマニュアルページ。 |

## ブラウザーの対応

{{Compat("api.WebGLRenderingContext.useProgram")}}

## 関連項目

- {{domxref("WebGLRenderingContext.createProgram()")}}
- {{domxref("WebGLRenderingContext.deleteProgram()")}}
- {{domxref("WebGLRenderingContext.isProgram()")}}
- {{domxref("WebGLRenderingContext.linkProgram()")}}
- {{domxref("WebGLRenderingContext.validateProgram()")}}
- {{domxref("WebGLRenderingContext.getProgramParameter()")}}
- {{domxref("WebGLRenderingContext.getProgramInfoLog()")}}
