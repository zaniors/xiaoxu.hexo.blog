---
title: 使用WebGL绘制一个漂亮的三角形
date: 2019-06-15 09:40:27
tags: [webgl, glsl, javascript]
categories: [前端, webgl]
thumbnail:
---

绘制一个三角形前，先看下GPU的渲染管线简图
![GPU绘制流水线简图](https://cdn.compelcode.com/image/fe/gpu-rendering-pipeline.jpg)
而现代的显卡已经支持开发者自己实现算法操作顶点和片元这两个模块，也就是我们说的可编程的着色器，GLSL。
> 上面流水线用三角形举例：

1. 顶点数据：一组包含三角形坐标顶点数据，像这样[0.0, 0.5, -0.5, -0.5, 0.5, -0.5]
2. 顶点着色器：处理顶点数据，位置，颜色等
3. 图元装配：会对我们整个场景成像裁剪，毕竟视域是有限的，超越裁剪体外就不可见；裁剪体针对图元进行的，这时会将顶点数据生成三角形这样的图元
4. 光栅化：由于裁剪的图元还只是顶点数据，这时必须确定哪些像素位于三角形内部，会生成帧缓存，而帧缓存就包含了每个片元携带的信息(颜色,位置等)
5. 片元着色器：更新片元信息
6. 屏幕成像

### 创建canvas与初始化webgl上下文
``` js 
var canvasEl = document.createElement('canvas')
canvasEl.setAttribute('id', 'glCanvas');
document.body.appendChild(canvasEl);

canvasEl.width = document.body.clientWidth;
canvasEl.height = 700;

// 早期可能会看到写成这样 var gl = canvasEl.getContext('webgl') || canvasEl.getContext('experimental-webgl');
// 那是因为早期webgl在试验阶段时获取webgl上下文的方式
var gl = canvasEl.getContext('webgl');  

gl.clearColor(1, 0.5, 0.5, 3);
gl.clear(gl.COLOR_BUFFER_BIT);
```
主要是说下clearColor和clear
- gl.clearColor：以什么颜色值(rgba)来清除颜色缓冲区。一般和clear一起使用。
- gl.clear：清空指定预设值缓冲区。指定gl.COLOR_BUFFER_BIT，清空颜色缓冲区

### 创建三角形的顶点GLSL代码
``` js
var vertexShaderSource = 
[
    'precision mediump float;',
    'attribute vec2 trianglePotision;',
    'void main() {',
        'gl_Position = vec4(trianglePotision, 0.0, 1.0);',
    '}'
].join('\n');
```

### 创建三角形的片元GLSL代码
``` js 
var fragmentShaderSource = 
[
    'percision mediump float;',
    'void main() {',
        'gl_FragColor = vec4(0.0, 0.0, 0.0, 1,0);',
    '}',
].join('\n');
```
> precision mediump float; // GPU在计算浮点数使用的精度
有三种类型：并不是所有硬件都支持highp，为了保证正常工作，尽量使用mediump/lowp

- highp：高等精度
- mediump：中等精度
- lowp：低等精度

### 目前的GLSL代码是无法工作的，需要创建shader实例、编译shader、生成shader一系列，它像下面这样：
``` js
var vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertexShaderSource);
gl.compileShader(vertexShader);

var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragmentShaderSource);
gl.compileShader(fragmentShader);
```
可以简单封装下：
``` js 
function createShader(gl, type, sourceCode) {
    var shader = gl.createShader(type);
    gl.shaderSource(shader, sourceCode);
    gl.compileShader(shader);
    
    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        throw 'error compiling shader：' + gl.getShaderInfoLog(shader);
    }
    return shader;
}

var vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
var fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);
```
> gl.getShaderParameter(shader, gl.COMPILE_STATUS)：获取着色器信息，配合COMPILE_STATUS来检查指定shader是否编译成功

### 我们的着色器已经创建，接下来创建我们的webgl program和我们的shader绑定，并且连接和使用
``` js 
// 生成一个webgl program
gl.program = gl.createProgram();
// 和shader绑定
gl.attachShader(gl.program, vertexShader);
gl.attachShader(gl.program, fragmentShader);

gl.linkProgram(gl.program);
// 检查指定program是否连接成功
if (!gl.getProgramParameter(gl.program, gl.LINK_STATUS)) {
    throw 'error linking program：' + gl.getProgramInfoLog(gl.program);
}

// 检查指定program是否正常工作
gl.validateProgram(gl.program);
if (!gl.getProgramParameter(gl.program, gl.VALIDATE_STATUS)) {
    throw 'error validating program：' + gl.getProgramInfoLog(gl.program);
}

gl.useProgram(gl.program);
```
> WebGL给开发者提供很多检查的Parameter

### 使用缓冲区(Buffer)缓存数据
缓冲区buffer用来缓存顶点数据和相关信息，可以传递多个顶点坐标供着色器使用
- 创建缓冲区对象
- 绑定缓冲区
- 写入缓冲区数据
- 将缓冲区数据写入顶点着色器
- 启用缓冲区

``` js
var triangleVertices = new Float32Array([
    // x y
    0.0, 0.5,
    -0.5, -0.5,
    0.5, -0.5,
]);

var triangleVertexBufferObject = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexBufferObject);
gl.bufferData(gl.ARRAY_BUFFER, triangleVertices, gl.STATIC_DRAW);

var trianglePositionLocation = gl.getAttribLocation(gl.program, 'trianglePotision');
gl.vertexAttribPointer(
    trianglePositionLocation,
    2,
    gl.FLOAT,
    0,
    0,
    0
);
gl.enableVertexAttribArray(trianglePositionLocation);
```

### 渲染图形
``` js
function render() {
    gl.drawArrays(gl.TRIANGLES, 0, 3);
}

render();
```
### 效果图
![](https://cdn.compelcode.com/image/fe/webgl-triangle-1.png)

### 绘制颜色
> 上面将三角形的位置通过变量传入着色器，下面将颜色也传入着色器

- 修改着色器代码
- 修改顶点数据，将颜色值也放入Buffer缓冲区
- 取着色器变量，赋值buffer颜色数据
- 启用缓冲区对象

#### 修改着色器
``` js 
var vertexShaderSource =
    [
        'precision mediump float;',
        'attribute vec2 trianglePotision;',
        'attribute vec3 triangleColor;',
        'varying vec3 fragColor;',
        'void main() {',
            'gl_Position = vec4(trianglePotision, 0.0, 1.0);',
            'fragColor = triangleColor;',
        '}'
    ].join('\n');

var fragmentShaderSource =
    [
        'precision mediump float;',
        'varying vec3 fragColor;',
        'void main() {',
            'gl_FragColor = vec4(fragColor, 1.0);',
        '}',
    ].join('\n');
```
attribute：用于顶点着色器，向顶点着色器传数据
varying：用于顶点着色器和片元着色器两者之间通信，需要两者同样定义

#### 修改缓冲区数据
``` js 
var triangleVertices = new Float32Array([
    // x y      r g b
    0.0, 0.5,   1.0, 1.0, 0.0,
    -0.5, -0.5, 0.7, 0.0, 1.0,
    0.5, -0.5,  0.1, 1.0, 0.6,
]);
```

#### 取顶点着色器里的颜色变量
``` js 
var triangleColorLocation = gl.getAttribLocation(gl.program, 'triangleColor');

gl.vertexAttribPointer(
    trianglePositionLocation,
    2,
    gl.FLOAT,
    0,
    5 * triangleVertices.BYTES_PER_ELEMENT,
    0
);
gl.vertexAttribPointer(
    triangleColorLocation,
    3,
    gl.FLOAT,
    0,
    5 * triangleVertices.BYTES_PER_ELEMENT,
    2 * triangleVertices.BYTES_PER_ELEMENT
);
```
由于缓冲区顶点数据改变，之前的位置信息也改变了
vertexAttribPointer：告诉显卡从当前绑定的缓冲区中读取数据
- 参数1(index)：将数据传值的顶点属性。
- 参数2(size)：每次取的数据大小。位置每次取两个，颜色则每次取三个。
- 参数3(type)：数据占多大字节。
- 参数4(normalized)：是否标准化。
- 参数5(stried)：每个顶点数据所占字节数。比如有这么一组数据[1, 0, 0, 1, 0, 1]，如果stried为0，然后每次取两个数据，最后则是(1, 0),(0, 1),(0, 1)；如果stried为3*4个字节，顶点数据每12个字节才能读到我们需要的数据，最后则是(1, 0),(1, 0)
- 参数6(offset)：偏移多少字节后开始读数据

#### 启用缓冲区对象
``` js
gl.enableVertexAttribArray(triangleColorLocation);
```

### 效果图
![](https://cdn.compelcode.com/image/fe/webgl-triangle-2.png)

<!-- ### 矩阵变换平移
![](https://cdn.compelcode.com/image/fe/webgl-translation-matrix.png) -->
