---
title: webgl基本介绍
date: 2019-06-02 12:16:58
tags: [webgl, glsl, javascript]
categories: [前端]
thumbnail:
---

WebGL是什么：它是浏览器上对OpenGL ES2.0标准的JavaScript接口实现，因为当时的OpenGL被设计成本地运行，无法发挥Web的优势。而WebGL标准是OpenGL ES2.0的一个JavaScript绑定，这样浏览器可以显示3D模型和场景的能力。由于WebGL也只是一部分的OpenGL实现，OpenGL ES2.0是基于着色器(Shader)的，还有部分核心需要通过GLSL(OpenGL Shading Language)语言开发，比如：视图转换、投影转换等，也就是JS代码中要编写GLSL才能实现

## 通过WebGL绘制一个点

### 创建一个webgl上下文(不是canvas 2d)
``` js
var glCanvas = document.getElementById('glCanvas');
var gl = glCanvas.getContext('webgl');
```

### 创建一个着色器程序
``` js
gl.program = gl.createProgram();
```

### 绑定着色器
``` js
gl.attachShader(gl.program, vShader);  // 绑定顶点着色器
gl.attachShader(gl.program, fShader);  // 绑定片元着色器
```
> 这里的vertexShader和fragmentShader这两个着色器都还没创建，见下面

### 连接program
``` js
gl.linkProgram(gl.program);
```

### 使用program
``` js
gl.useProgram(gl.program);
```

以上是创建一个webgl program的整个过程，题文也说了，WebGL也只是对OpenGL一部分的实现，接下来的顶点和片元着色器就需要通过GLSL来写

### 创建一个顶点着色器
> 顶点着色器：用来描述顶点属性(比如位置)，让GPU计算最终顶点位置，输出至gl_Position。它可以是1个顶点(Point)、2个顶点(Lines)、3个顶点(Triangles)
``` js
var VSHADER_SOURCECODE = 
'void main() {\n' +
    'gl_Position = vec4(0.0, 0.0, 0.0, 1.0);\n' +
    'gl_PotinSize = 10.0;\n' +
'}\n';
var vShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vShader, VSHADER_SOURCECODE);
gl.compileShader(vShader);
```

### 创建一个片元着色器
> 片元着色器：用来处理物体材质，灯光等信息，也可理解为像素
``` js
var FSHADER_SOURCECODE = 
'void main() {\n' +
    'gl_FlagColor = vec4(1.0, 0.0, 0.0, 1.0);\n' +
'}\n';
var fShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fShader, FSHADER_SOURCECODE);
gl.compileShader(fShader);
```

### 绘制
``` js
gl.clearColor(0.0, 0.0, 0.0, 1.0);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.POINTS, 0, 1);
```

### 整理及优化
``` js
var glCanvas = document.getElementById('glCanvas');
var gl = glCanvas.getContext('webgl');

gl.program = gl.createProgram();

initShader();

gl.linkProgram(gl.program);

gl.useProgram(gl.program);

render();

function initShader() {
    var vShader, fShader;
    var VSHADER_SOURCECODE, FSHADER_SOURCECODE;

    VSHADER_SOURCECODE = 
    'void main() {\n' +
        'gl_Position = vec4(0.0, 0.0, 0.0, 1.0);\n' +
        'gl_PotinSize = 10.0;\n' +
    '}\n';
    FSHADER_SOURCECODE = 
    'void main() {\n' +
        'gl_FlagColor = vec4(1.0, 0.0, 0.0, 1.0);\n' +
    '}\n';

    vShader  = createShader(gl, VSHADER_SOURCECODE, gl.VERTEX_SHADER);
    fShader = createShader(gl, FSHADER_SOURCECODE, gl.FRAGMENT_SHADER);

    gl.attachShader(gl.program, vShader);
    gl.attachShader(gl.program, fShader);
}

function createShader(context, sourceCode, type) {
    var shader = context.createShader(type);
    context.shaderSource(shader, sourceCode);
    context.compileShader(shader);
    return shader;
}

function render() {
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.POINTS, 0, 1);
}
```