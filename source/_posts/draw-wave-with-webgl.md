---
title: 使用WebGL绘制波浪效果
date: 2019-08-03 13:59:43
tags: [webgl, glsl, javascript]
categories: [前端, webgl]
thumbnail:
---

以前接触过的是利用canvas来绘制一些粒子效果，这篇文章通过webgl一步一步绘制圆点波浪效果

### 绘制一个point
1. 创建webgl/canvas
2. 创建shader
3. 创建webgl program
4. 绘制一个point圆点
5. 绘制多个point圆点
6. 让圆点动起来

#### 创建webgl/canvas
>  webgl借助opengl虽然具有绘制3D的能力以及调用GPU硬件加速等，但是最终还是需要canvas对象来呈现，它们的关系：

![渲染关系](https://cdn.compelcode.com/image/fe/webgl-and-cavans.jpg)
``` html
  <canvas id="glcanvas" width="700" height="500"></canvas>
```
``` js
  const canvasEl = document.body.querySelector('#glcanvas')
  const gl = canvasEl.getContext('webgl');

  gl.clearColor(0, 0, 0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
```

#### 创建shader
> 编写GLSL，让底层去编程GPU的渲染管线的一部分
``` js
  // 顶点着色器
  // gl_Position：描述位置信息
  // gl_PointSize：绘制一个点的大小
  const V_SHADER_SOURCE =
  [
    'void main() {',
      'gl_Position = vec4(0.0, 0.0, 0.0, 1.0);',
      'gl_PointSize = 10.0;',
    '}'
  ].join('\n');

  // 片元着色器
  // gl_FragColor：描述颜色信息
  const F_SHADER_SOURCE = 
  [
    'void main() {',
      'gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);',
    '}',
  ].join('\n');
```

#### 创建WebGL Program，编译shader，program和shader绑定
``` js
  var program = gl.createProgram();
  var vShader = createShader(gl, gl.VERTEX_SHADER, V_SHADER_SOURCE);
  var fShader = createShader(gl, gl.FRAGMENT_SHADER, F_SHADER_SOURCE);
  gl.attachShader(program, vShader);
  gl.attachShader(program, fShader);

  gl.linkProgram(program);
  // program连接失败抛出异常
  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
    throw '\n' + 'error linking program：' + gl.getProgramInfoLog(program);
  }
  gl.useProgram(program);

  function createShader(gl, type, shaderSource) {
  var shader = gl.createShader(type);
  gl.shaderSource(shader, shaderSource);
  gl.compileShader(shader);

  // 由于在JS以字符串写的GLSL，很多错误只能在编译后才能知道
  // 需要借助gl.getShaderParameter来捕获后，抛出异常信息
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    throw '\n' + 'error compiling shader：' + gl.getShaderInfoLog(shader);
  }

  return shader;
}
```

#### 绘制point
``` js
  render();
  function render() {
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.POINTS, 0, 1);
  }
```

#### 效果图
![绘制一个point](images/webgl/webgl-point.png)

不过这并不是我想要的效果，我想要绘制一个圆点，并不是正方形
#### 绘制一个point圆点

![gl_PointCoord](images/webgl/webgl-gl_PointCoord.png)

1. 利用片元着色器提供的变量gl_PointCoord，来访问当前片元所在图元的二维坐标(0.0至1.0)，如上图：
2. 将以中心点(0.5, 0.5)的距离超过0.5的话，利用discard将之外片元忽略

``` js 
const F_SHADER_SOURCE =
  [
    'precision mediump float;',
    'void main() {',
      'float diff = distance(gl_PointCoord, vec2(0.5, 0.5));',
        'if (diff <= 0.5) {',
          'gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);',
        '} else {',
          'discard;',
        '}',
    '}',
  ].join('\n');
```
> discard：只可用于片元着色器，当控制流遇到这个关键字时，正在处理的片元就会被标记为将要丢弃
#### 圆点效果图
![绘制一个point](images/webgl/webgl-circle.png)

#### 着色器数据动态化
- attribute：顶点着色器使用，传输与顶点着色器相关数据
- uniform：顶点或者片元着色器使用，传输与非顶点着色器数据，比如颜色，如果顶点与片元着色器申明相同的uniform，那么将会共享
- varying：顶点着色器向片元着色器传递数据，两个着色器都必须要申明相同的varying(包括变量名与类型都必须一致)
``` js
  // 修改顶点着色器
  const V_SHADER_SOURCE =
  [
    'attribute float v_point_size',
    'void main() {',
      'gl_Position = vec4(0.0, 0.0, 0.0, 1.0);',
      'gl_PointSize = v_point_size;',
    '}'
  ].join('\n');

  const vPointSize = gl.getAttribLocation(program, 'v_point_size');
  gl.vertexAttrib1f(vPointSize, 20.0)

  // 渲染
  render()
```
- 既然这样，是不是可以循环，然后渲染出动态效果
``` js
  let i = 0;
  let flag = false
  setInterval(() => {
    gl.vertexAttrib1f(aPointSize, i);
    
    if (i === 40) flag = true
    if (i === 0) flag = false

    flag ? i-- : i++;
    draw()
  }, 10);
```
#### 效果图
![绘制一个point](images/webgl/point-ani.gif)

比如说要渲染大量的point，或者point的位置从A移动到B，当然通过循环实现没毛病，但我们需要大量数据绘制时，可以利用WebGL提供的缓存区对象（buffer data），缓存区对象存放大量的数据供着色器使用
> 同理，将gl_Position和gl_FragColor，也可以存放变量里，不同的是，片元着色器要用uniform申明，获取则gl.uniformLocation，设置则gl.uniform4f
不管是设置顶点还是片元，webgl都提供多种方式vertexAttrib[1234]f[v]、uniform[1234][uif][v]，具体参考：
1. [设置uniform文档](https://developer.mozilla.org/en-US/docs/Web/API/WebGL2RenderingContext/uniform)
2. [设置vertex文档](https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/vertexAttrib)

#### 绘制多个point圆点
利用webgl缓冲区，让显卡从缓冲区读取顶点着色器数据
- 创建buffer
- 绑定buffer
- 定义buffer数据，写入缓冲区
- 通过vertexAttribPointer将指定规则的数据，写入指定的顶点属性变量中，最后启用指定缓存数据

``` js
  const aPointSize = gl.getAttribLocation(program, 'a_point_size')
  const aPointPosition = gl.getAttribLocation(program, 'a_position')

  const bufferData = new Float32Array([
    // point_size
    10.0, 20.0, 30.0,

    // point_potision x y z
    -0.3, -0.3, 0.0,
    0.0, 0.0, 0.0,
    0.3, 0.3, 0.0,
  ])

  const vertexBuffer = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer)
  gl.bufferData(gl.ARRAY_BUFFER, bufferData, gl.STATIC_DRAW)

  gl.vertexAttribPointer(aPointSize, 1, gl.FLOAT, 0, 0, 0)
  gl.enableVertexAttribArray(aPointSize)

  gl.vertexAttribPointer(aPointPosition, 3, gl.FLOAT, 0, 0, 3 * bufferData.BYTES_PER_ELEMENT)
  gl.enableVertexAttribArray(aPointPosition)

  // draw()
  gl.drawArrays(gl.POINTS, 0, 3)
  // draw参数：
  // 1、mode：绘制的形状，gl.POINTS，gl.LINES，gl.TRIANGLES，当然还有其它可以参考文档
  // 2、first：从哪个点开始
  // 3、count：绘制多少次
```
[vertexAttribPointer详情用法参考之前文章](https://compelcode.com/post/draw-triangle-with-webgl#%E5%8F%96%E9%A1%B6%E7%82%B9%E7%9D%80%E8%89%B2%E5%99%A8%E9%87%8C%E7%9A%84%E9%A2%9C%E8%89%B2%E5%8F%98%E9%87%8F)
已经完成了如何使用缓冲区，并且将缓冲区数据写入顶点着色器中

![利用缓冲区绘制point](images/webgl/webgl-point-1.png)

#### 动态生成缓冲区数据

当然我们要实现的是"波浪效果"，包含上百个这样的point，不可能手动去写
在生成"波浪"数据缓冲区之前，先看看webgl的坐标与canvas坐标的区别与如何转换
##### webgl默认使用的是右手坐标系，如下图：

![Right-Handed Coordinate System](images/webgl/right-handed-coordinate-system.webp)
- canvas的0,0点在左上角，Y向下，坐标值为屏幕的像素
- webgl的0,0点在canvas画布的中间，整个webgl尺寸区间在-1至1

##### 将canvas坐标转换到webgl：

![canvas-to-webgl](images/webgl/canvas-to-webgl.png)

#### 让圆点动起来