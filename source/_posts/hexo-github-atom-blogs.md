---
title: 使用hexo，github，atom搭建个人博客
tags:
  - 基础工具
categories:
  - 其他技术
date: 2018-12-19 16:40:55
---


### hexo的配置与使用
##### hexo的基础用法
* hexo的安装
    ``` bash
    $ brew install node
    $ npm install -g hexo-cli
    ```

* hexo博客站点搭建
    ``` bash
    $ hexo init blog  # 创建名为blog的博客站点
    $ cd blog
    $ hexo new [draft/post/page] name  # 新建名为name的草稿/文章/页面，默认为post
    $ hexo g & hexo s  # 生成静态网页并启动本地服务器，之后便可以通过http://localhost:4000/访问
    ```

##### hexo的主题设置
> 推荐[_next_](https://theme-next.iissnan.com)主题，简洁且经典
1. 下载theme到`/path/to/blog/themes`目录下；
2. 修改`/path/to/blog/_config.yml`文件中的`theme`字段为要设置的theme的目录名；
3. `hexo g & hexo s`重新生成静态文件预览；

### 用git平台作免费空间
##### 配置github仓库
1. 在github创建一个Repository Name与用户名一致的项目；
2. 安装`hexo-deployer-git`插件：`npm install hexo-deployer-git --save`;
3. 修改`_config.yml`文件的deploy属性：
    ```
    deploy:
      type: git
      repo:
        github: https://github.com/rabintang/rabintang.github.io
      branch: master
    ```
4. 发布文章到github，运行`hexo g & hexo d`即可；

##### 配置[coding平台](https://coding.net)仓库
1. 跟github类似，创建一个Repository Name与用户名同名的项目；
2. 修改`_config.yml`文件的中deploy属性：
    ```
    deploy:
      type: git
      repo:
        github: https://github.com/rabintang/rabintang.github.io
        coding: git@git.coding.net:rabintang/rabintang.git
      branch: master
    ```

##### 配置域名访问规则
> 原则：国外IP访问博客站点时，用github作为站点提供服务，国内IP访问博客站点时，用coding作为站点提供服务。

{% asset_img 万网配置.png 万网配置 %}

##### 原始文件保存与多机同步
> 方法1：将生成的静态页发布到github/coding平台博客项目的`master`分支，将源文件同步到github的`hexo`分支。
> 方法2：创建一个新的github项目，专门用来存放hexo的源文件；
> 因为方法2比较简单，因此这里只介绍方法1的详细做法，方法2类似。

1. 关联github项目
    ```
    cd /path/to/blog
    git init
    git remote add origin https://github.com/xx/xx.github.io
    ```
2. 创建`hexo`分支并提交
    ```
    git checkout -b hexo
    git add .
    git commit -m 'add hexo'
    git push origin hexo
    ```
3. 修改github项目的主分支为`hexo`：_Settings -> Branchs -> Default branch_
4. 其他机器初始化
    ```
    git clone https://github.com/xx/xx.github.io
    cd xx.github.io
    git checkout hexo
    npm install
    ```

### hexo写作技巧
##### 在文章中插入图片
* 通过同级的资源文件目录：

    将 `_config.yml` 文件中 `post_asset_folder` 设置为 `true`，即可以在创建文章时，在同级目录创建一个资源文件夹。通过`![](image.jpg)`即可在文章中插入资源文件夹中的图片。

    但是上面相对路径的方式，只能在文章页正确显示图片，要想在首页等其他页面也正确显示图片，要采用标签插件语法：`{% asset_img image.jpg This is an image %}`。

* 通过全局资源文件目录：

    创建`source/images`文件夹，将图片存放在该文件夹，在文章中通过`![](/images/image.jpg)`来访问图片路径，即可以在所有地方都正确显示图片。

* 通过CDN存放资源文件：

    当然也可以通过CDN来存放图片等资源文件，推荐 [极简图床](https://jiantuku.com/) 和 [七牛云](https://www.qiniu.com) 来搭建免费的图床CDN。极简图床提供了chrome插件，方便图片的上传和管理。

##### 写作流程
> draft -> post
1. 创建草稿：`hexo new draft blog_name`；
2. 发布草稿：`hexo publish blog_name`；

##### 管理分类与标签
###### 分类管理
1. 创建cateogries页面：`hexo new page categories`；
2. 修改`/path/to/blog/source/categories/index.md`内容如下：
    ```
    ---
    title: 分类
    date: 2018-11-10 11:19:48
    type: "categories"
    comments: false
    ---
    ```
3. 给文章添加categories属性：
    ```
    ---
    title: *******
    categories:
      - category1
      - category2
    ---
    ```
    注意hexo每篇文章只会属于一个类目，如上面设置两个category，则会将文章归到`category1 / category2`这样的目录结构，其中，`category2`是`category1`的子目录。

###### 标签管理
1. 创建tags页面：`hexo new page tags`;
2. 修改`/path/to/blog/source/tags/index.md`内容如下：
    ```
    ---
    title: 标签
    date: 2018-11-10 11:23:25
    type: "tags"
    comments: false
    ---
    ```
3. 给文章添加tags属性：
    ```
    ---
    title: *******
    tags:
      - tag1
      - tag2
    ---
    ```
    不同于categories，每篇文章可以属于多个tags。

### 用atom作为写作工具
##### atom tricks汇总
* 控制面板快捷键：`shift+cmd+p`

##### atom的插件推荐
* markdown-preview-enhanced：对atom默认的markdown进行了强化，提供了不少有用功能，网上也有不少文件推荐各种markdown插件的组合，但是感觉还是安装这一个插件方便，毕竟入门的话定制化不需要那么强；
* platformio-ide-terminal：在atom的底部显示terminal窗口，方便在当前目录执行命令，打开terminal窗口快捷键`shift+cmd+t`，关闭terminal窗口快捷键`shift+cmd+x`；
* markdown-writer：暂时没有用到；
* simplified-chinese-menu：atom汉化插件，给有需要的人；
