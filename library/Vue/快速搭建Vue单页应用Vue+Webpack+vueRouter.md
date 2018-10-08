# 快速搭建Vue单页应用Vue+Webpack+vueRouter

1. ##### Vue.js简介
   官方解释：    
   Vue.js (读音 /vjuː/，类似于 view) 是一套构建用户界面的渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，它不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与单文件组件和 Vue 生态系统支持的库结合使用时，Vue 也完全能够为复杂的单页应用程序提供驱动。
2. ##### 安装NPM
   参考：[npm 使用介绍](http://www.runoob.com/nodejs/nodejs-npm.html)    
   npm太慢解决办法：[淘宝npm镜像使用方法](https://npm.taobao.org/)
3. ##### 安装vue-cli
   1.在确认npm成功安装的情况下，开始安装vue-cli。Vue-cli是vue官方出品的快速构建单页应用的脚手架，其作用是帮助用户尽可能快地开始编写实际的应用程序。    
   2.通过 npm 命令安装vue-cli（安装淘宝镜像后可以尝试用cnpm来进行安装），在命令行（最好以管理员身份打开命令行）输入下面的命令：
   ```java
   /* -g :代表全局安装 */
   npm install vue-cli -g
   ```
   3.安装完成后，可以用vue -V来进行查看 vue-cli的版本号。如果vue -V命令能显示版本号，说明已经顺利安装好了vue-cli。
4. ##### 初始化项目
   1.我们可以用vue init命令来初始化项目：
   ```java
   $ vue init <template-name> <project-name>

   /**
    init：表示我要用vue-cli来初始化项目

    <template-name>：表示模板名称，vue-cli官方为我们提供了5种模板，

    webpack-一个全面的webpack+vue-loader的模板，功能包括热加载，linting,检测和CSS扩展。

    webpack-simple-一个简单webpack+vue-loader的模板，不包含其他功能，让你快速的搭建vue的开发环境。

    browserify-一个全面的Browserify+vueify 的模板，功能包括热加载，linting,单元检测。

    browserify-simple-一个简单Browserify+vueify的模板，不包含其他功能，让你快速的搭建vue的开发环境。

    simple-一个最简单的单页应用模板。

    <project-name>：标识项目名称，这个你可以根据自己的项目来起名字。
   */
   ```
   通常我们选用webpack这个模板,所以在命令行输入以下命令：
   ```java
   $ vue init webpack vuecliDemo

   /**
   输入命令后，会询问我们几个简单的选项，我们根据自己的需要进行填写就可以了。

    Project name :项目名称 ，如果不需要更改直接回车就可以了。注意：这里不能使用大写，所以我把名称改成了vueclidemo
    Project description:项目描述，默认为A Vue.js project,直接回车，不用编写。
    Author：作者，如果你有配置git的作者，他会读取。
    Install  vue-router? 是否安装vue的路由插件，我们这里需要安装，所以选择Y
    Use ESLint to lint your code? 是否用ESLint来限制你的代码错误和风格。我们这里不需要输入n，如果你是大型团队开发，最好是进行配置。
    setup unit tests with  Karma + Mocha? 是否需要安装单元测试工具Karma+Mocha，我们这里不需要，所以输入n。
    Setup e2e tests with Nightwatch?是否安装e2e来进行用户行为模拟测试，我们这里不需要，所以输入n。
   */
   ```
   2.如果命令行出现文字“ vue-cli · Generated "vuecliDemo" ”,说明已经初始化好了第一步。    
   3.接下来在命令行运行下面的命令来安装我们的项目依赖包，也就是安装package.json里的包，如果你网速不好，你也可以使用cnpm来安装。
   ```java
   $ npm install
   ```
   4.最后通过下面的命令来在开发模式下运行我们的程序。
   ```java
   npm run dev  //dev表示开发模式
   ```
   如果能成功打开应用，说明已经安装成功。    

   详情参考：    
   1.[Webpack3.X使用介绍](http://jspang.com/2017/09/16/webpack3-2/)    
   2.[Vue-cli教程](http://jspang.com/2017/04/10/vue-cli/)
5. ##### 项目打包
   1.在命令行中输入npm run build命令后，vue-cli会自动进行项目发布打包。
   ```java
   npm run build
   ```
   在执行完npm run build命令后，会在项目根目录生成dist文件夹，这个文件夹里边就是我们要传到服务器上的文件。主要包含：    
   1.index.html 主页文件:因为我们开发的是单页web应用，所以说一般只有一个html文件。    
   2.static 静态资源文件夹：里边js、CSS和一些图片。
