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
4. 绘制point

#### 创建webgl/canvas
>  webgl借助opengl虽然具有绘制3D的能力以及调用GPU硬件加速等，但是最终还是需要canvas对象来呈现，它们的关系：

![渲染关系](http://cdn.compelcode.com/image/fe/webgl-and-cavans.jpg)
``` js
  const canvasEl = document.body.appendChild(document.createElement('canvas'));
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
  draw();
  function draw() {
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.POINTS, 0, 1);
  }
```

#### 效果图
![绘制一个point](images/webgl/webgl-point.png)

不过这并不是我想要的效果，我想要绘制一个圆点，并不是正方形
#### 绘制圆点
> 利用片元着色器提供的变量gl_PointCoord，来访问图片坐标