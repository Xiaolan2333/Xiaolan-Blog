---
title: "使用Hexo搭建博客并部署至Github Pages"
date: 2025-03-01 12:00:00
---

## 环境准备
安装 Node.js 和 Git
Node.js：下载地址: https://nodejs.org
Git：下载地址: https://git-scm.com/downloads/win
 <!--more--> 

## 安装 Hexo
全局安装 Hexo CLI
找一个顺眼的文件夹，右键-Open Git Bash here
输入：
```Bash
npm install -g hexo-cli
```
注：若网络环境较差，可尝试换源，在Git Bash窗口中执行：
```Bash
npm config set registry https://registry.npmmirror.com
 ```


## 初始化 Hexo 项目
创建博客目录并初始化，在刚才的Git窗口中分别输入并执行：
```Bash
hexo init blog
```
```Bash
cd blog
```
```Bash
npm install
```


## 本地运行测试
在Git窗口中输入并执行
hexo s
访问： http://localhost:4000 查看效果


## 创建新文章
在Git窗口中输入并执行
hexo new 文章标题
编辑生成在 \source\_posts 内的 文章标题.md`文件，添加内容


## 生成静态文件
在Git窗口中输入并执行
```Bash
hexo g
```


## 部署到 GitHub Pages
步骤 1：创建 GitHub 仓库
仓库名必须为 你的GitHub用户名.github.io
仓库类型必须为Public
创建仓库后，在 https://github.com/用户名/用户名.github.io/settings/pages 内打开Github Pages

步骤 2：安装部署插件
在Git窗口中输入并执行：
```Bash
npm install hexo-deployer-git --save
```

步骤 3：配置 _config.yml
在文件末尾添加：
```_config.yml
deploy:
  type: git
  repo: https://github.com/你的用户名/仓库名.git
  branch: main
```

步骤 4：部署到 GitHub
在Git窗口中输入并执行
hexo d
首次部署可能需要输入 GitHub 账号密码
成功后访问 https://你的用户名.github.io 查看博客。


## 常用命令

新建文章 hexo new 标题
清理缓存 hexo clean
生成静态文件 hexo g
本地预览 hexo s
一键部署 hexo d


## 常见问题
1.页面未更新：执行 hexo clean && hexo g && hexo d ；
2.主题不生效：确认 _config.yml 的缩进和语法正确。

完成以上步骤后，你的博客应该已经成功部署到 GitHub Pages