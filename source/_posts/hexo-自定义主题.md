---
title: hexo 自定义主题
date: 2022-04-21 10:28:41
tags: hexo
cover: /img2/butterfly.png
copyright_author: hong
copyright_author_href: 
copyright_url: https://louishongyi.github.io/icarus.github.io/2022/04/21/use-hexo-and-github-to-create-you-blog/
copyright_info: 此文章版权归 hong 所有，如有转载，请註明来自原作者
---



选中的自定义主题为 ： [Butterfly](https://github.com/jerryc127/hexo-theme-butterfly)

如果直接应用改主题， 像头像啊，url, 文字啊什么的，这些需要自定义的就不能修改。所以需要自行更改项目。

<!-- more -->

1. 将该项目 fork 为自己的项目

2. 在自己的博客git 仓库，运行：

   ```bash
   $ git submodule add <https://github.com>:LouisHongYi/hexo-theme-butterfly themes/Butterfly
   ```

   ![](1.png)

   然后项目就被定义为模块放在 `thems` 文件夹下

   ![](2.png)

3. 更改 `_comfig.yml` 中的  `theme: butterfly` 。这个就是主题的名字。

4. 更改 `theme` 下的 `Butterfly`项目，推送该子模块，然后回到项目主目录 `hexo clean g d` 上传至git。

主题推荐：

[Next](http://theme-next.iissnan.com/getting-started.html) 。应用广泛，自定义很多。

[Butterfly](https://butterfly.js.org/posts/21cfbf15/) 。自己应用的第一个主题。
