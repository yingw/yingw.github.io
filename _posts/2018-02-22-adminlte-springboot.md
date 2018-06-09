---
layout: article
title: Spring Boot 和 AdminLTE 的集成
tags: SpringBoot AdminLTE
key: 2018-02-22-adminlte-springboot
---

# Spring Boot 和 AdminLTE 的集成

AdminLTE 是个开源的管理模板，首页：http://adminlte.io/，GitHub地址：https://github.com/almasaeed2010/AdminLTE，目前有 2w+ start，100 多个 contributor，可以说是非常出名的开源管理模板了。

![image](https://adminlte.io/img/AdminLTE2.1.png)

这篇描述将 AdminLTE 和 Spring Boot 整合起来进行后台开发并利用 AdminLTE 的前端组件、模板、布局。

- 使用的 AdminLTE 版本 v2.4.3 （2018-02-22）
- 项目地址：https://github.com/yingw/adminlte-boot-template

下面将介绍三种前后端集成的方式
- static：所有 AdminLTE 的静态资源直接集成到项目
- webjars：用 [Webjars](https://www.webjars.org/) 将前端库集成到后端 maven 的依赖 jar 中
- bower：用 [Bower](https://bower.io/) 将前端依赖定义，在项目编译过程中创建前端资源

## 初步集成

### 1. 创建 Spring Boot 项目
从 http://start.spring.io/ 或者 IDE 中创建一个 Spring Boot 项目，依赖：Web、Thymeleaf、DevTools

### 2. 下载 AdminLTE
从 AdminLTE 的 [GitHub release 页](https://github.com/almasaeed2010/AdminLTE/releases) 下载项目（v2.4.3），解压

- 拷贝 pages 目录、index.html、index2.html、start.html 到 `templates` 目录
- 拷贝 dist 下 css、img、js 目录、bower_components、plugins 目录到 `static` 目录
> 注意的是 bower_components 内有上千文件，拷贝、编译都很慢，虽然用后面的 webjars 依赖可以避免，目前还是需要的，在项目代码中没有放，请自行拷贝。

### 3. 修正路径

批量替换 pages 下面页面的一些相对路径，主要是 css、js、img 的引用
- `../../dist/` 替换为 `../../`
- `../dist/` 替换为 `../`
- `"dist/` 替换为 `"./`
- 还有一些上级目录的如 index.html 替换

### 4. 动态跳转控制器
创建一个动态跳转控制器 AdminController
```java
/**
 * @author yinguowei@gmail.com 2018/3/27.
 */
@Controller
class AdminController {
    private static Logger logger = LoggerFactory.getLogger(AdminController.class);

    @GetMapping("/")
    public String home() {
        return "redirect:index2.html";
    }

    @GetMapping(value = {"/**/*.html"})
    public String route(HttpServletRequest request) {
        logger.debug("AdminController.route: request.getRequestURI() = {}", request.getRequestURI());
        String path = request.getRequestURI();
        return path;
    }
}
```

设置些属性 application.properties
```property
spring.thymeleaf.cache=false
spring.thymeleaf.suffix=
```

设置了这个控制器后所有的 `.html` 就可以跳转到对应的视图。如果将来不希望通过 .html 后缀访问 url，而是采用类似 restful 的地址，可以去掉 `spring.thymeleaf.suffix` 定义（默认是 `.html`），并在解析时候去掉 .html
```java
    @RequestMapping(value = {"/**/*.html"})
    public String route(HttpServletRequest request) {
        logger.debug("DynamicController.route: request.getRequestURI() = {}", request.getRequestURI());
        String path = request.getRequestURI();
        return path.substring(1, path.length() - 5); // remove "/" and ".html"
    }
```

### 5. 字体和主题
#### 临时去掉 Googleapi 字体
由于 googleapi 的字体可能被墙，建议全部注释

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Source+Sans+Pro:300,400,600,700,300italic,400italic,600italic">
```

#### 加上雅黑字体
并在主题内加上
AdminLTE.css 和 AdminLTE.min.css
```css
font-family: 'Microsoft YaHei UI', 'Source Sans Pro', 'Helvetica Neue', Helvetica, Arial, sans-serif;
```
 来使用客户端的**微软雅黑**字体

#### 主题替换
如果需要主题，定制专门的 skin 和 skin.min.css

替换
  `<link rel="stylesheet" href="/css/skins/_all-skins.min.css">`
  为
  `<link rel="stylesheet" href="dist/css/skins/skin-blue.min.css">`

#### 生成 min.css

TBD，有在线工具

### 其他修正

部分页面由于 json/javascript 格式问题和 Thymeleaf 冲突，比如 charts/flot.html 改成：

<html xmlns:th="http://www.thymeleaf.org">

```html
<script th:inline="none">
```

### 静态测试

Thymeleaf 的一大好处就是可以直接打开 html 进行页面测试而不需要启动服务器。编辑各 stylesheet 和 javascript 的应用地址，改为和当前页面相对，如：
```html
<link rel="stylesheet" href="../static/bower_components/bootstrap/dist/css/bootstrap.min.css" th:href="{/bower_components/bootstrap/dist/css/bootstrap.min.css}">
<script src="../static/bower_components/jquery/dist/jquery.min.js" th:src="@{/bower_components/jquery/dist/jquery.min.js}"></script>
```
>但注意这一步也可以不用急着改太多页面，而是只改一两个测试下，因为后面要放到 Layout 公共页面统一改。


### 小结

经过这一系列集成，已经可以跑起来测试 http://localhost:8080，会自动跳转到 index2.html，可能有些路径的修正不完善或有问题，观察控制台 404 异常修正。

## Thymeleaf Layout

Layout 是让多个页面共用一套页面模板，这样很多重复的代码就不用重复编码了。

引入 pom 依赖
```xml
<dependency>
  <groupId>nz.net.ultraq.thymeleaf</groupId>
  <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

在 layout 目录创建 `layout.html`，可以用 index2.html 拷贝过去修改

### layout/layout.html

标题
```
  <title layout:title-pattern="$LAYOUT_TITLE | $CONTENT_TITLE">AdminLTE 2</title>
```

修改原来的相对地址，去掉 static，只保留大部分页面都要用的如 bootstrap, jquery, datatable

```html
  <!-- Bootstrap 3.3.7 -->
  <link rel="stylesheet" href="/bower_components/bootstrap/dist/css/bootstrap.min.css">
  <!-- Font Awesome -->
  <link rel="stylesheet" href="/bower_components/font-awesome/css/font-awesome.min.css">
  <!-- Ionicons -->
  <link rel="stylesheet" href="/bower_components/Ionicons/css/ionicons.min.css">
  <!-- jvectormap -->
  <link rel="stylesheet" href="/bower_components/jvectormap/jquery-jvectormap.css">
  <!-- Theme style -->
  <link rel="stylesheet" href="/css/AdminLTE.min.css">
  <!-- AdminLTE Skins. Choose a skin from the css/skins
       folder instead of downloading all of them to reduce the load. -->
  <link rel="stylesheet" href="/css/skins/_all-skins.min.css">
```

修改菜单可以动态根据传入变量选中
```html
<li class="treeview" th:classappend="${menu=='index' || menu=='index2' ? 'active menu-open' : ''}">
  <ul class="treeview-menu">
    <li th:classappend="${menu=='index' ? 'active' : ''}"><a href="/index.html"><i class="fa fa-circle-o"></i> Dashboard v1</a></li>
```

修改 img 的相对地址，如
```html
<img class="direct-chat-img" src="../static/img/user1-128x128.jpg" th:src="@{/img/user1-128x128.jpg}" alt="message user image">
```

js
```html
<!-- jQuery 3 -->
<script src="/bower_components/jquery/dist/jquery.min.js"></script>
<!-- Bootstrap 3.3.7 -->
<script src="/bower_components/bootstrap/dist/js/bootstrap.min.js"></script>
<!-- AdminLTE App -->
<script src="/js/adminlte.min.js"></script>

<!-- Optionally -->
<!-- Slimscroll -->
<script src="/bower_components/jquery-slimscroll/jquery.slimscroll.min.js"></script>
<!-- FastClick -->
<script src="/bower_components/fastclick/lib/fastclick.js"></script>
<!-- AdminLTE for demo purposes -->
<script src="/js/demo.js"></script>

<div layout:fragment="customScript" th:remove="tag">

</div>

```

如果想单独测试 layout.html，跟下面 index2 一样修改加上静态和动态的地址

#### 声明插入点
```html
<div class="content-wrapper" layout:fragment="content">
...
<div layout:fragment="customScript" th:remove="tag">
</tag>
```

### index2.html

修改 index2.html、start.html、blank.html 来使用 layout

#### 头
```html

<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{layout/layout.html}"
      th:with="menu='index2'">
```

#### stylesheet 和 javascript

为了本地测试，保留 stylesheet 和 javascript，但是加上 `th:remove="all"` 在解析时自己删掉

```html
  <!-- Bootstrap 3.3.7 -->
  <link rel="stylesheet" href="../static/bower_components/bootstrap/dist/css/bootstrap.min.css" th:remove="all">
  <!-- Font Awesome -->
  <link rel="stylesheet" href="../static/bower_components/font-awesome/css/font-awesome.min.css" th:remove="all">
  <!-- Ionicons -->
  <link rel="stylesheet" href="../static/bower_components/Ionicons/css/ionicons.min.css" th:remove="all">
  <!-- Theme style -->
  <link rel="stylesheet" href="../static/css/AdminLTE.min.css" th:remove="all">
  <!-- AdminLTE Skins. -->
  <link rel="stylesheet" href="../static/css/skins/_all-skins.min.css" th:remove="all">
  <!-- jvectormap -->
  <link rel="stylesheet" href="../static/bower_components/jvectormap/jquery-jvectormap.css" th:remove="all">
  <!-- 表格 -->
  <link rel="stylesheet" href="../static/bower_components/datatables.net-bs/css/dataTables.bootstrap.min.css" th:remove="all">
  <!-- PACE -->
  <link rel="stylesheet" href="../static/bower_components/PACE/themes/blue/pace-theme-minimal.css" th:remove="all">

```

js，页面自定义的 js 可以放到 customScript fragment 里面去
```html
<!-- jQuery 3 -->
<script src="../static/bower_components/jquery/dist/jquery.min.js" th:remove="all"></script>
<!-- Bootstrap 3.3.7 -->
<script src="../static/bower_components/bootstrap/dist/js/bootstrap.min.js" th:remove="all"></script>
<!-- FastClick -->
<script src="../static/bower_components/fastclick/lib/fastclick.js" th:remove="all"></script>
<!-- AdminLTE App -->
<script src="../static/js/adminlte.min.js" th:remove="all"></script>

<div layout:fragment="customScript">

  <!-- Sparkline -->
  <script src="../static/bower_components/jquery-sparkline/dist/jquery.sparkline.min.js" th:src="@{/bower_components/jquery-sparkline/dist/jquery.sparkline.min.js}"></script>
  <!-- jvectormap  -->
  <script src="../static/plugins/jvectormap/jquery-jvectormap-1.2.2.min.js" th:src="@{/plugins/jvectormap/jquery-jvectormap-1.2.2.min.js}"></script>
  <script src="../static/plugins/jvectormap/jquery-jvectormap-world-mill-en.js" th:src="@{/plugins/jvectormap/jquery-jvectormap-world-mill-en.js}"></script>
  <!-- SlimScroll -->
  <script src="../static/bower_components/jquery-slimscroll/jquery.slimscroll.min.js" th:src="@{/bower_components/jquery-slimscroll/jquery.slimscroll.min.js}"></script>
  <!-- ChartJS -->
  <script src="../static/bower_components/chart.js/Chart.js" th:src="@{/bower_components/chart.js/Chart.js}"></script>
  <!-- AdminLTE dashboard demo (This is only for demo purposes) -->
  <script src="../static/js/pages/dashboard2.js" th:src="@{/js/pages/dashboard2.js}"></script>
  <!-- AdminLTE for demo purposes -->
  <script src="../static/js/demo.js" th:src="@{/js/demo.js}"></script>

</div>

```


#### 删掉不用的节点

删掉：`navbar-custom-menu`、`main-sidebar`（也可以保留菜单测试）、`main-footer`、`control-sidebar`、`control-sidebar-bg`

#### 内容声明
在 content-wrapper 上声明
```html
<div class="content-wrapper" layout:fragment="content">
```

### login layout

还有些 404、500 页面使用的是和 layout 完全不同的布局，也可以提炼一个 `layout-login`，然后 login.html、logout.html、404、500 等都使用这个布局，这里就不重复了。

### 总结

通过使用 layout，可以将大部分页面共用代码提取出来，通过 fragment 插入页面的定制内容。但是这里大部分页面将来都不会是项目内必须使用的，就不全部改了。

## Webjars

上面介绍的基本就是静态集成的全部内容，不过前面提到，要把 bower_componenets 这个几千个文件夹的第三方 js、css 依赖库放到项目管理中，总是不太方便的，对于后端开发来说，最好前端的组件也是几个依赖来管理就最简单了，webjars 就是以这个目标设计的工具。

### Webjars 依赖

根据项目使用的库的版本，到 webjars 官网去搜索对应的定义即可，再把 maven 定义拷进来。

例如：
```xml
<!--bootstrap-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>bootstrap</artifactId>
	<version>3.3.7</version>
</dependency>
```


### 替换 bower_components 为 /webjars

可以直接从 /webjars 开始应用这些依赖，如：
```html
<link rel="stylesheet" href="/webjars/bootstrap/3.3.7/dist/css/bootstrap.min.css">
```
>注意个别有大小写问题：Ionicons，等等

有些 plugins 目录的依赖也可以通过 webjars 改掉，可能是作者稍微做了修改，如 pace

注意将来要是用上 Spring Security，记得把 webjars 上下文的资源设置为可以访问
```java
..., "/webjars/**").permitAll();
```

```xml
<!-- Webjars begin -->
<!--bootstrap-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>bootstrap</artifactId>
	<version>3.3.7</version>
</dependency>
<!--font-awesome-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>font-awesome</artifactId>
	<version>4.7.0</version>
</dependency>
<!--ionicons-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>ionicons</artifactId>
	<version>2.0.1</version>
</dependency>
<!--morrisjs-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>morrisjs</artifactId>
	<version>0.5.1</version>
</dependency>
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>raphael</artifactId>
	<version>2.2.7</version>
</dependency>
<!--sparkline-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>jquery-sparkline</artifactId>
	<version>2.1.3</version>
</dependency>
<!--jvectormap-->
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>jvectormap</artifactId>
	<version>2.0.4</version>
</dependency>
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>bower-jvectormap</artifactId>
	<version>1.2.2</version>
</dependency>
<!--datepicker-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>bootstrap-datepicker</artifactId>
	<version>1.7.1</version>
</dependency>
<!--daterangepicker-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>bootstrap-daterangepicker</artifactId>
	<version>2.1.27</version>
</dependency>
		<!--?? https://github.com/bootstrap-wysiwyg/bootstrap3-wysiwyg 0.3.3 -->
		<!--<dependency>-->
		<!--<groupId>org.webjars.bower</groupId>-->
		<!--<artifactId>bootstrap3-wysihtml5-bower</artifactId>-->
		<!--<version>0.3.3</version>-->
		<!--</dependency>-->
<!--moment-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>moment</artifactId>
	<version>2.20.1</version>
</dependency>
<!--knob-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>jquery-knob</artifactId>
	<version>1.2.13</version>
</dependency>
<!--chartjs-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>chartjs</artifactId>
	<version>1.0.2</version>
</dependency>
<!--icheck-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>iCheck</artifactId>
	<version>1.0.2</version>
</dependency>
<!--Flot.js-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>flot</artifactId>
	<version>0.8.3</version>
</dependency>
<!--color picker-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>bootstrap-colorpicker</artifactId>
	<version>2.5.1</version>
</dependency>
<!--select2-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>select2</artifactId>
	<version>4.0.5</version>
</dependency>
<!--ckeditor-->
<dependency>
	<groupId>org.webjars.npm</groupId>
	<artifactId>ckeditor</artifactId>
	<version>4.8.0</version>
</dependency>
		<!--fullcalendar-->
		<!--<dependency>-->
			<!--<groupId>org.webjars.npm</groupId>-->
			<!--<artifactId>fullcalendar</artifactId>-->
			<!--<version>3.8.2</version>-->
		<!--</dependency>-->
<!--PACE (update from original plugins-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>pace</artifactId>
	<version>1.0.2</version>
</dependency>

<!--datatables-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>datatables.net</artifactId>
	<version>1.10.16</version>
</dependency>
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>datatables.net-bs</artifactId>
	<version>2.1.1</version>
</dependency>
<!--		<dependency>
            <groupId>org.webjars</groupId>
            <artifactId>datatables-plugins</artifactId>
            <version>1.10.16</version>
        </dependency>
        <dependency>
            <groupId>org.webjars.bower</groupId>
            <artifactId>datatables.net-buttons</artifactId>
            <version>1.4.0</version>
        </dependency>
        <dependency>
            <groupId>org.webjars.bower</groupId>
            <artifactId>datatables.net-buttons-bs</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.webjars.bower</groupId>
            <artifactId>datatables.net-select</artifactId>
            <version>1.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.webjars.bower</groupId>
            <artifactId>datatables.net-select-bs</artifactId>
            <version>1.2.2</version>
        </dependency>-->
<!--fastclick-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>fastclick</artifactId>
	<version>1.0.6</version>
</dependency>
<!--jquery-->
<dependency>
	<groupId>org.webjars.bower</groupId>
	<artifactId>jquery</artifactId>
	<version>3.3.1</version>
</dependency>
<!--jquery-ui-->
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>jquery-ui</artifactId>
	<version>1.11.4</version>
</dependency>
<!--layer-->
<!--<dependency>-->
<!--<groupId>org.webjars.bower</groupId>-->
<!--<artifactId>github-com-sentsin-layer</artifactId>-->
<!--<version>3.0.3</version>-->
<!--</dependency>-->
<!--metisMenu-->
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>metisMenu</artifactId>
	<version>2.7.0</version>
</dependency>
<!--slimScroll-->
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>jQuery-slimScroll</artifactId>
	<version>1.3.8</version>
</dependency>
<!--pace (updated from Plugins)-->
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>pace</artifactId>
	<version>1.0.2</version>
</dependency>
<!-- switchery for checkbox -->
<!--<dependency>-->
<!--<groupId>org.webjars.bower</groupId>-->
<!--<artifactId>switchery</artifactId>-->
<!--<version>0.8.2</version>-->
<!--</dependency>-->
<!-- sweetalert -->
<dependency>
	<groupId>org.webjars.npm</groupId>
	<artifactId>sweetalert</artifactId>
	<version>2.1.0</version>
</dependency>

<!-- Webjars end -->
```
### Webjars Locator

还有个 Webjars 的 Locator 可以用来自动检索版本，写引用 url 的时候就可以连版本号也省略了。

但是会没有代码提示功能，而且只在版本需要经常升级不想动代码的时候才有用，意义不大。

```xml
<!--webjars-locator-->
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>webjars-locator</artifactId>
	<version>0.30</version>
</dependency>
```

### 使用 webjars 的限制

使用 webjars，也有缺点，就是在静态测试的时候，没法直接应用 jar 内的样式和脚本，所以用 webjars 的时候就不能做静态测试了；好处是不用在代码库中存着那上千个依赖。

## Bower

最后一种方式，还可以使用 Bower 来在编译时生成依赖文件。需要用到 maven-frontend-plugin，安装 node 和 bower

AdminLTE 本身就用了 bower 来管理依赖，拷贝项目中的 bower.json 到根目录，创建 .bowerrc


## 使用本项目

clone 后，由于 master 分支是基于静态依赖，但是没有提交 bower_components 目录。可以：

1. 第一种方式自己下载 AdminLTE 后解压该目录到 static 目录
2. 第二种方式直接用 webjars 分支
3. 第三种方式先切换到 bower 分支，编译完成就自动下载在 static 目录了，再切换回 static 目录或者就使用 bower 分支

个人建议用 webjars 分支，如果对 node、bower 很熟悉也不在乎超多文件打包就使用 bower 分支。