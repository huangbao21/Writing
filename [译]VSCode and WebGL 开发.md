[译]VSCode and WebGL 开发

> [原文链接](https://medium.com/@armno/vscode-and-webgl-development-dfc17bba52ed)
> </br>**作者: Armno P.TH**
> </br>**理由: 简单实用的小伎俩**

### Foreword
Vscode 是一个优秀的前端开发编辑器，但仍然会有一些问题的存在，我觉得我应该做一些笔记。
</br></br> 我见过大部分教程都是以这样开头: 首先创建`index.html` and `main.js`文件

- 在`index.html`放入`<canvas>`

```html
<canvas id="canvas"></canvas>
```
在`main.js`，获取 canvas 元素来作为渲染 WebGL内容的上下文

```js
const canvas = document.getElementById('canvas');
// or const canvas = document.querySelector('#canvas');
const gl = canvas.getContext('webgl');
```
- 然后基本上所有东西都使用`gl`对象

```js
// methods
gl.clearColor(0.0, 0.0, 0.0, 1.0);
gl.attachShader();
gl.vertexAttrib1f();
gl.getAttribLocation();
// constants
gl.COLOR_BUFFER_BIT;
gl.VERTEX_SHADER;
gl.COMPILE_STATUS;
```
我发现VSCode并不能智能提示有关`gl`代码，因为VSCode发现它是 `any` 类型，或者VSCode不知道`gl`对象实际到底是什么类型。因此我无法使用编辑器的智能提示功能

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\1.png)

刚开始我觉得它没什么问题，可能也会忍受。但你会发现，`gl`对象是`WebGLRenderingContext`类型，并且如果去查阅这个接口规范，会发现有超过200个属性(常量)和方法。我的天，我将他们全部记住可能性为 0 好么？！而且超过87.525%的几率我将打错这些属性和方法。

### 如何从VSCode中得到相应的辅助呢？

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\2.gif)
实际造成这个的原因在于VSCode不知道`canvas`元素 从`document.getElementById()`or `document.querySelector()`方法中返回的是什么类型。
<br/>使用`document.getElementById()`,`HTMLElement`类型将被返回

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\3.png)
使用`document.querySelector()`,`Element`将被返回

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\4.png)
然而两种都不够具体，我们需要更明确的类型来表示canvas - `HTMLCanvasElement`。我认为有2种变通的方法可以使用

### Option1：使用document.createElement()
不需要在HTML中创建`<canvas>`标签，而是直接用JavaScript创建它，并动态的放入页面中

```js
const canvas = document.createElement('canvas');
document.querySelector('body').appendChild(canvas);
const gl = canvas.getContext('webgl');
```
通过`document.createElement('canvas')`创建canvas，VSCode能知晓它是属于`HTMLCanvasElement`类型，并且通过调用`.getContext()`在canvas上，VSCode能得到它实际类型，不再是`any`

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\5.png)
传入一个字符串`webgl`作为参数，VSCode知道方法将返回`WebGLRenderingContext`类型
![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\6.png)
现在我们可以获得代码的自动提示了

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\7.png)
### Option2：使用TypeScripts Type Assertion(类型断言)
如果项目使用TS，我们能用 Type Assertion 来告诉VSCode，canvas对象是属于什么类型

```
const canvas = <HTMLCanvasElement>document.getElementById('canvas');
```
这告诉TS编译器将`canvas`对象作为`HTMLCanvasElement`类型，而不是默认`HTMLElement`类型。但是它和类型转换还不一样。
<br/>当VSCode知道类型后，它将可以像下面那样代码自动提示。

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\8.png)
### 作为学习资源的代码编译器
VSCode有了智能提示功能后是非常有帮助的。不仅可以帮助我们加快编码速度减少错误率，还可以作为一个工具来学习WebGL新的方法和APIs，不需要额外去查官方文档
<br/>例如，鼠标悬停在一个方法名上将会提示方法的参数，顺序，和返回的值。要不然每次都要在浏览器和google中切换来查看第五个参数是什么

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\9.png)
在方法或者一个属性名上点击`Cmd+click(or Ctrl+click)`将跳转到定义上。如果幸运的话，定义文件里将有相关文档。虽然WebGL APIs还没完全实现，但是我相信它最终会像其他API一样内置

![](D:\f\github\Writing\imgs\[译]VSCode and WebGL 开发\10.png)

### 补充:安装webpack 和 TypeScript
如果在项目中使用TS，则少不了转译成JS这一步。所以使用webpack进行构建转译。简单示例如下：
1. 安装TS

```node
npm install --save-dev typescript
```
2. 安装webpack
```
npm install --save-dev webpack webpack-cli webpack-dev-server
```
3.安装TS loader并支持sourcemaps

```
npm install --save-dev awesome-typescript-loader source-map-loader
```
4. 创建`webpack.config.js`配置文件

```js
const path = require('path');
module.exports = {
  mode: 'development',
  entry: './src/index.ts',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
 },
 devServer: {
   contentBase: './dist'
 },
 resolve: {
   extensions: ['.ts', '.tsx', '.js', '.jsx']
 },
 devtool: 'source-map',
 module: {
   rules: [
     {
       test: /\.tsx?$/,
       loader: 'awesome-typescript-loader'
     }
   ]
 }
};
```
5. 启动webpack服务

```
npx webpack-dev-server
```
END~