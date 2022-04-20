---
title: title
date: 2022-04-20 15:55:39
tags: blog
---

# hexo  github

1. 安装最新的 `Node.js`

   -需要使用最新的 `node.js` 版本，否则安装 `hexo` 可能会报错。

2. 安装git

3. 本地git 关联github, 并赋权

4. 创建仓库

- 仓库务必以 `[github.io](<http://github.io>)`  结尾。

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/121d1373-b2c5-478e-88fa-39cc4527b797/Untitled.png)

1. 到新建仓库， `Pages` 选择 branch， choose  a theme, and save.

   ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/878e5cea-c70b-4847-bb0a-c4310445907b/Untitled.png)

2. 在自己电脑新建文件夹，右键 `Git Bash Here` 依次执行：

   1. 安装 hexo `npm install hexo-cli -g`
   2. 验证 `hexo -v`
   3. 初始化 `hexo init`
   4. 安装前端组件 `npm install`
   5. 编译 `hexo g`
   6. 启动服务 `hexo s` , 成功后不关闭 git bash 窗口

3. 访问 [`http://localhost:4000/`](http://localhost:4000/)

4. 更改 `_config.yml` 文件

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

5. 上传到 github , 然后就可以随时更改了。

   - 清理 `hexo clean`
   - 编译 `hexo g`
   - git 上传

   ```sql
   -- 全部文件加入暂存区
   git add -A .
   git commit -m 'init hexo'
   git push -u origin main
   ```

6. 打开你的仓库地址。 就是本地的  [`http://localhost:4000/`](http://localhost:4000/)

   ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7a9272b8-d4bd-49db-935c-6fedefe5efc4/Untitled.png)

7. 写文章

   1. 安装扩展 `npm install hexo-deployer-git`

   2. 新建文章 `hexo new post "use hexo and github to create you blog"`

   3. 然后你会发现 `source\\_posts` 文件夹下有了两个新的md文件，打开刚刚新建的md, 编辑你的文章

      ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/af3a2e00-cdc7-4fa6-82e4-6febfdbe73ce/Untitled.png)

   4. 编辑好了以后

      1. 编译 `hexo g` 生成静态文件
      2. 上传至git
      3. 打开  https://louishongyi.github.io/icarus.github.io/



###  

