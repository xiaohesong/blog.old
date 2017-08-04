---
title: Build Hexo
date: 2017-08-04 13:58:49
tags: hexo
---


# 动手搭建基于Hexo的Blog


## 基本操作

### 依赖

- [node的依赖.](https://github.com/creationix/nvm#install-script)

### 安装hexo

```bash
mkdir <github-name>.github.io -p
cd <github-name>.github.io
sudo npm install -g hexo-cli
```

### 初始化博客框架

```bash
hexo init
npm install
```

### 初次展现
经过上面简单的步骤,已经把基本的博客框架给搭建好了.可以查看下效果.

```bash
hexo s
```
然后输入 `localhost:4000`可以查看下基本的效果.
不过,这不是我想要的.


## 美化操作

### 主题选择

下载主题到`themes`

```bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

这样就把主题搞定了.不过要配置下.在`<github-name.github.io>`的`_config.yml`中找到`theme`.
```bash
theme: next #注意, 默认是landscape
```
这里可配置文件选项较多,[请查看文档](http://theme-next.iissnan.com/getting-started.html)

#### [更多配置](http://theme-next.iissnan.com/)

## 注意事项

#### `git push`到`github page`之后出现404.邮件反馈:
- First
```html
The page build completed successfully, but returned the following warning for the `master` branch:

You are attempting to use a Jekyll theme, "next", which is not supported by GitHub Pages. Please visit https://pages.github.com/themes/ for a list of supported themes. If you are using the "theme" configuration variable for something other than a Jekyll theme, we recommend you rename this variable throughout your site. For more information, see https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/.

For information on troubleshooting Jekyll see:

  https://help.github.com/articles/troubleshooting-jekyll-builds

If you have any questions you can contact us by replying to this email.
```

这个是要指定下, [可以参考](https://hexo.io/docs/deployment.html).

- Second

```html
The page build failed for the `master` branch with the following error:

The tag `fancybox` on line 77 in `themes/landscape/README.md` is not a recognized Liquid tag. For more information, see https://help.github.com/articles/page-build-failed-unknown-tag-error/.

For information on troubleshooting Jekyll see:

  https://help.github.com/articles/troubleshooting-jekyll-builds

If you have any questions you can contact us by replying to this email.
```

这个也比较好解决的.去`themes`下的`landscape`删除`README.md`文件.
