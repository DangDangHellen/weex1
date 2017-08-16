# Weex快速上手
Weex是一套简单易用的跨平台开发方案，能以web的开发体验构建高性能、可扩展的native应用，为了做到这些，Weex与Vue合作，使用Vue作为上层框架，并遵循W3C标准实现了统一的JSEngine和DOM API，这样一来，你甚至可以使用其他框架驱动Weex，打造三端一致的native应用。

## 搭建环境
使用dotWe对Weex尝鲜是一个不错的选择，但是如果你想更专业的开发Weex，dotWe就不怎么够用了。下来我们搭建本地环境进行Weex开发。

### 安装依赖
请确保已安装了node环境。然后安装weex-toolkit。
	
	node -v
	npm -v
	npm install -g weex-toolkit
	
最后通过weex命令验证是否安装成功。

### 初始化项目
初始化一个weex项目：
	weex init first-weex-project
	
命令行里出现如下结果：
![](/Users/hellen/Desktop/test.png)










进入到first-weex-project目录下，weex-toolkit已经为我们生成了标准项目结构，目录列表如下：
![](/Users/hellen/Desktop/test1.png)

### 开发
在package.json中已经配置好了几个常用的npm脚本，分别是：
	
	"scripts": {
    "build": "webpack",               	                          源码打包
    "dev": "webpack --watch",                                     webpack watch模式，方便开发    
    "serve": "webpack && node build/init.js && serve -p 8080",    开启静态服务器
    "debug": "weex-devtool"。                                     调试模式
  }
  
首先来通过npm install安装项目依赖。之后运行npm run dev 和 npm run serve开启 watch模式和静态服务器。然后打开浏览器，进入http://localhost:8080/index.html 即可看到Weex H5页面。
![](/Users/hellen/Desktop/result.png)





## 分析文件

由上面的开发项目目录，我们主要的开发是在src文件夹里。初始项目中默认有一个foo.vue文件，这相当于一个组件。然后在app.js中引入了这个组件，并且将其挂载点设置为#root。而#root是weex.html文件下的一个id为root的div元素。weex.html文件是在index.html作为框架iFrame被引入的。

这样一条线下来，就可以知道，入口页面是index.html，然后是如何一步步引入到我们自定义组件上，并展示相应的内容的。那么下来我们主要关心的就是foo.vue组件是如何写的？


主要是分成三个部分来写的。`<template> </template>`是HTML模版，用来写HTML结构；`<style></style>`用来写样式；`<script></script>`用来写脚本。这里的语法跟Vue语法是相同的。了解过Vue的应该都知道，页面是由数据来渲染的，所以script标签中主要是数据的定义，以及方法的定义。如果HTML中要引用数据，必须用{{}}来括起来；而如果定义元素属性，则不需要{{}}。看下初始化项目中foo.vue的代码：
![Foo.png](/Users/hellen/Desktop/Foo.png)


## 自定义组件

打算将所有组件放在一个文件夹里，所以在src文件夹下再创建一个components文件夹。我们自己写的组件就可以放在这个文件夹里面。

比如我们要将原先foo.vue中的内容显示多个，而又不想重复在foo.vue里面去写。可以将其内容抽出来作为一个组件ChildFoo，然后在foo.vue里加入多个ChildFoo就可以了。那么ChildFoo组件如何去实现？注意：***自组件里的data必须是个函数，返回一个对象（键值对的格式）***，原因是为了保证各个组件里的数据是各自一份，而不是共有的一份，这也是Vue语法中规定的。
所以ChildFoo里的代码与Foo唯一的不同在于 data的不同。

既然把页面显示的内容已经封装到组件ChildFoo里面了，Foo中必然不需要之前的那些代码了，那么Foo如何去引用我们的自定义组件ChildFoo呢？这里用过import ChildFoo from './components/ChildFoo.vue'来引入，注意：***一定要加后缀.vue，不然会报错***。同时在export default{ }中加入一个components项，其对应一个对象，这里面要写你要导出的组件。所以新的foo和ChildFoo代码是这样的：
![修改版Foo](/Users/hellen/Desktop/fooVue.png)
![ChildFoo代码](/Users/hellen/Desktop/ChildFoo.png)


这时候，npm run serve一下，就可以看到展示了两个相同的部分。
![添加组件后](/Users/hellen/Desktop/result1.png)

### 组件间通信

如果我还想让两个组件相同的结构，但是展示不同的图片，不同的标题，该怎么办呢？这时候我们可以把数据提出来，放到父组件里，然后将不同的数据传给自组件就可以了。在Vue中，子组件是通过props属性来获取父组件传的参数的。但是在父组件需要注意，***传参时需要用v-bind:来绑定数据，也可以简写成:参数***，类似于下面这样：
	
	<ChildFoo :source="logoUrl" :title="title"></ChildFoo>
	
如果有更多相同的组件，可以利用Vue的列表渲染来实现。

	<div v-for='item in sourceArr'>
      <ChildFoo :source="item.logoUrl" v-bind:title="item.title"></ChildFoo>
    </div>

如果子组件想去改变父组件中某个数据的值，子组件里面是不允许对props中的参数进行修改的。因此可以利用`this.$emit()`通知父组件去进行相应操作。

	this.$emit('onClick', {title: 'Weex'});

另一方面，父组件要去监听子组件触发的这个函数

	v-on:onClick="handleClick($event)"
handleClick就是自定义的事件处理函数，其中$event是事件对象，包含了从子组件传过来的对象参数。
最终得到下面的效果：
![](/Users/hellen/Desktop/Weex.png)

最后，说一说如何在移动端看页面效果。手机下载PlayGround，[下载地址](http://weex.apache.org/cn/playground.html)，利用playground扫码就可以了。


## 参考内容：
1. [Weex官网](http://weex.apache.org/cn/guide/index.html)
2. [Vue官网](https://cn.vuejs.org/v2/guide/)
3. [Weex组件间通信 简书](http://www.jianshu.com/p/3f61b0a1a530)
4. [Weex小例子-WeexOne](https://github.com/dodola/WeexOne)















