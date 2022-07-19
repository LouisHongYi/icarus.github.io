---
title: 使用 hexo 和 github 搭建博客
date: 2022-04-21 10:28:41
tags: hexo
cover: /img2/hexo.png
copyright_author: hong
copyright_author_href: 
copyright_url: https://louishongyi.github.io/icarus.github.io/2022/04/21/use-hexo-and-github-to-create-you-blog/
copyright_info: 此文章版权归 hong 所有，如有转载，请註明来自原作者
---



使用 `hexo` 在 `github` 上搭建自己的博客。

[`hexo` 官方文档。](https://hexo.io/zh-cn/docs/)

1. 安装最新的 `Node.js`

   -需要使用最新的 `node.js` 版本，否则安装 `hexo` 可能会报错。

2. 安装git

3. 本地git 关联github, 并赋权
   <!-- more -->

4. 创建仓库

   - 仓库务必以 `[github.io](<http://github.io>)`  结尾。

   ![](create-repository.png)

5. 到新建仓库， `Pages` 选择 branch， choose  a theme, and save.

   ![](set-pages.png)

6. 在自己电脑新建文件夹，右键 `Git Bash Here` 依次执行：

   1. 安装 hexo `npm install hexo-cli -g`
   2. 验证 `hexo -v`
   3. 初始化 `hexo init`
   4. 安装前端组件 `npm install`
   5. 编译 `hexo g`
   6. 启动服务 `hexo s` , 成功后不关闭 git bash 窗口

7. 访问 [`http://localhost:4000/`](http://localhost:4000/)

8. 更改 `_config.yml` 文件

   - 一共有两处

   ```sql
   -- 修改url , 你的仓库地址
   # URL
   ## Set your site url here. For example, if you use GitHub Page, set url as '<https://username.github.io/project>'
   url: <https://louishongyi.github.io/icarus.github.io>
   permalink: :year/:month/:day/:title/
   permalink_defaults:
   pretty_urls:
     trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
     trailing_html: true # Set to false to remove trailing '.html' from permalinks
   
   -- 修改 type, repository, branch 
   # Deployment
   ## Docs: <https://hexo.io/docs/one-command-deployment>
   deploy:
     type: git
     repository: <https://github.com/LouisHongYi/icarus.github.io>
     branch: main
   ```

9. 打开你的远程仓库地址。 本地的  [`http://localhost:4000/`](http://localhost:4000/)

   ![](url.png)

10. 写文章

    1. 安装扩展 `npm install hexo-deployer-git`

    2. 新建文章 `hexo new post "use hexo and github to create you blog"`

    3. 然后你会发现 `source\\_posts` 文件夹下有了两个新的md文件，打开刚刚新建的md, 编辑你的文章

       ![](folder.png)

11. 上传到 github , 然后就可以随时更改了。

    - 清理 `hexo clean`
    - 编译 `hexo g` 生成静态文件
    - 上传git `hexo d`
