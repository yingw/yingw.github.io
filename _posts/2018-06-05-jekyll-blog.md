---
layout: article
title: 用 Jekyll 搭建个人博客
tags: Jekyll
key: 2018-06-05-jekyll-blog
---

# 用 Jekyll 搭建个人博客

一直计划用 Github Pages 和 Jekyll 搭建个人博客，把之前散在各博客：CSDN、ITEYE、博客园，以及自己各种笔记软件里的文章都迁移过来。直到最近做了个基于 Jekyll 的网站的翻译工作有了点基础。遂走起——

>注：整个建站过程需要用到以下工具或组件

- [Ruby](https://www.ruby-lang.org)
- [RubyGems](https://rubygems.org)
- [Bundle](https://bundler.io)
- [Jekyll](https://jekyllrb.com)
- [GitHub Pages](https://pages.github.com)
- [Jekyll Themes](http://jekyllthemes.org)
- [阿里云域名](https://wanwang.aliyun.com/domain)（可选）

## 安装 Ruby 环境

首先，Jekyll 是基于 Ruby 开发的，需要 Ruby 开发环境。根据[官网](https://www.ruby-lang.org/en/downloads/)的说法，Linux 和 Mac 直接用系统的包管理工具就可以安装 Ruby。Windows 的 Ruby 需要使用 **RubyInstaller** 来安装。

### RubyInstaller
前往 [https://rubyinstaller.org/downloads/]() 下载 RubyInstaller 并安装

> 注：需要 Ruby 2.1.0 以上版本

安装完成后检测一下 Ruby 是否正确安装及版本：
```sh
ruby -v
```

## 安装 RubyGems
Ruby 1.9.2 以上版本已经默认安装了RubyGems，就不需要再手动安装了。

> 注：需要 RubyGems 2.6 以上

检查版本：
```sh
gem -v
```

### 切换 Ruby 的国内源

虽然不用安装 RubyGems 了，但是在后续使用前建议把下载 Gem 的源切换为国内的 Gem 源，这里用的是 ruby-china 的源：
```sh
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
```

查看源
```
gem sources -l
```

## 安装 Bundle
```
gem install bundle
```

## 安装 Jekyll
```
gem install jekyll
```

## Jekyll Themes 模板
前往 [Jekyll Themes](http://jekyllthemes.org) 选一个想用的模板。clone 该模板的项目或者下载 zip 解压直接使用。比如这里我选了个 [**TeXt**](http://jekyllthemes.org/themes/TeXt/) 模板：

![](https://raw.githubusercontent.com/kitian616/jekyll-TeXt-theme/master/screenshots/TeXt-home.png)

```sh
git clone https://github.com/kitian616/jekyll-TeXt-theme.git
```
> 官网说了有`普通安装`和`主题安装`两种方式，这里介绍的是简单的普通安装，后续会介绍更灵活的主题安装。

其他优秀模板
- [https://gaohaoyang.github.io/]()
- [http://jekyllthemes.org/themes/jekyll-theme-next/]()
- [http://jekyllthemes.org/themes/simple-elegant/]()
- [https://github.com/iissnan/hexo-theme-next]()

### 安装 Gem 依赖
下载或克隆成功后，直接启动会有报错缺失 gem，先安装各依赖 Gem，进入项目目录：
```sh
bundle install
```

### 启动 Jekyll Server
```sh
bundle exec jekyll serve
```
![](https://raw.githubusercontent.com/yingw/yingw.github.io/master/assets/images/201806/20180605_jekyll_server.png)
启动成功后（大概一分钟），就可以访问：[http://localhost:4000]() 来使用模板了。

![](https://raw.githubusercontent.com/yingw/yingw.github.io/master/assets/images/201806/20180605_text_theme.png)

>启动如果有报错字符集，那是因为 windows 的命令行工具默认字符集不是 UTF8，可以先执行 `chcp 65001`，再启动就可以
```sh
chcp 65001
bundle exec jekyll serve
```

>如果启动有报错 `Permission denied - bind(2) for 127.0.0.1:4000`，那是因为 4000 端口被什么程序使用了，使用 netstat 找出是哪个进程占用了端口，关闭应用或杀掉进程：
```sh
# 找到进程 PID
netstat -aon|findstr "4000"
# 假设上面命令找到的 PID 是 207216
tasklist|findstr "207216"
# 得知是什么进程后可以关闭程序或者杀掉进程
taskkill /PID 207216
```
>后面两步也可以用任务管理器来定位 PID 和结束进程。
>我这里是被 FoxitReader 的一个保护进程占用了，查一下，杀掉 FoxitProtect 进程。

### TeXt 的一些设置
有关 TeXt 模板，作者已经在 [GitHub](https://github.com/kitian616/jekyll-TeXt-theme) 上写了很详细的中英文使用教程，我就简单提一下我用到的重点：
- 大部分配置信息都可以在 `_config.yml` 里面修改（但注意 _config 的内容改变后不会立即生效，需要重启 Jekyll）
- 可以删掉 `test`、`screenshots`、`docs` 目录
- 需要引用的图片可以放在 `assets/images/` 目录下，引用的时候如：`http://localhost:4000/assets/images/octocat.jpg`
- `_site` 是 Jekyll 运行时生成的网站代码，不要修改了这里的网页或者提交它们（已 gitignore）
- 可以设置 **LeanCloud**（看 TeXt 文档） 来使用点击量统计功能（注意要使用点击统计功能的话，md 文件头部的 key 字段不可省略）
- 可以设置 **Gitalk** 来使用留言
- 如果设置了时区启动报错，可以去 `Gemfile` 里把那句注释掉的 `gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw]` 释放出来，重新 `bundle install` 运行就行了
- 如上面截图，还有个 404 模板没找到的警告，暂时不知为何

## 开始写文章

接下来就可以使用模板并书写自己的文章了：

1. 在根目录下新建一个 `_post` 目录，
2. 然后就在里面添加 **MarkDown** 格式的文章
>Jekyll 要求文章的文件名格式都是：`年-月-日-标题.md`，比如这篇文章的文件名为：`2018-06-05-jekyll-blog.md`，将来就会自动发布到 `http://localhost:4000/2018/06/05/jekyll-blog.html` URL 上。
3. 在新建的 md 文件内最顶部添加内容：
```md
---
layout: article
title: 文章标题
tag: jekyll
key: 20180605_jekyll_blog
---
```
这一段是必要的，其中 `layout` 是文章使用的模板，不用修改，`title` 是指文章的标题。还可以加上 `tags` 来给文章设定标签用来分类检索（多个关键字用英文的空格分隔）。

再往下就可以开始写 **MarkDown** 格式的文章了，有关 MarkDown 的语法，可以参考[这篇教程](https://guides.github.com/features/mastering-markdown/)。

Jekyll 在运行过程中是实时同步网页的变更的，只要保存了 md 文件，就会自动在 _site 生成 html（注意看 Jekyll 的控制台 Log），所以保存好之后过个几秒，就可以看到效果了。

至此，本地的开发环境已经准备好了，就可以本地边编写边查看网站效果了，下面要把网站提交到 GitHub Pages 上去。

### 图片的相对路径

前面提到，图片存放的位置和访问的路径。但是为了之后同时在网站、GitHub 代码库、其他笔记中访问都能得到正确的图片路径（之前的相对路径可以满足前两个，但是外部笔记就没有相对路径的目录），可以把图片的相对路径改为发布到 GitHub 后的代码库路径，比如：
```markdown
![](/assets/images/201806/jekyll_server.png)
```
改为
```markdown
![](https://raw.githubusercontent.com/yourname/yourname.github.io/master/assets/images/201806/20180605_jekyll_server.png)
```

## 提交 GitHub Pages

首先创建一个名为 yourname.github.io 的新的仓库，再将所有代码和自己创建的 _posts 都提交过去就好了。具体怎么提交就不细说了，大致：
```sh
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/yingw/yingw.github.io.git
git push -u origin master
```

进入仓库的 Setting 查看一下 GitHub Pages 的设置是否已经在生成了。全部完成后访问：`https://yourname.github.io`，就能在线访问我们的博客了。

到这里开发和在线环境都已经配置好了，后续要做的就是坚持每天在 _posts 里面写新的文章并推送到 GitHub，一两分钟后就生效可以查看了。

## 域名绑定（可选）

如果有个人域名，或者不想用 *.github.io 这个域名，还可以自己申请一个并通过设置 CNAME 来跳转到 GitHub Pages 托管的网站上。

### 申请域名

可以用些免费的二三级域名，也可以花钱去阿里云申请一个：[https://wanwang.aliyun.com/domain]()

各种付款、审计通过后，在域名服务管理域名：[https://dc.console.aliyun.com/next/index#/domain/list/all-domain]()，点“解析” 进入 “解析设置”，添加一条解析设置：
- “记录类型” 为 “CNAME”
- “主机记录” 设置为 *
- “记录值” 就是 GitHub 的 yourname.github.io 域名

![](https://raw.githubusercontent.com/yingw/yingw.github.io/master/assets/images/201806/20180605_dns_setup.png)

最后回到我们的项目代码中，在根目录创建一个 CNAME 文件，文件内容就是一行上面申请的域名，来告诉 GitHub 运行这个域名引来的访问可以访问我们的空间。提交该文件。过一会等 GitHub Pages 编译好，就可以通过我们自己的域名访问博客了。

## 参考文章
- [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
- [Adding a Jekyll theme to your GitHub Pages site with the Jekyll Theme Chooser](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site-with-the-jekyll-theme-chooser/)
- [jekyll博客搭建之艰辛之路](https://segmentfault.com/a/1190000012468796)
- [Windows: No timezone data source could be found #1097](https://github.com/middleman/middleman/issues/1097)
- [TeXt 文档](https://tianqi.name/jekyll-TeXt-theme/docs/zh/quick-start)
- [Jekyll Quick Start Guide](https://jekyllrb.com/docs/quickstart/)