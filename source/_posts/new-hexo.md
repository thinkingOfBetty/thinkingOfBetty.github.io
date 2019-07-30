---
title: hexo 采坑记
---

查看了网上的教程，发现有些是部署到自己服务器的，有些是从安装git开始的，总是感觉不太适合自己，所以决定记录一个简历版本的hexo+githubpage的配合记录。

首先，你要在github上配置一个仓库，仓库名字也有讲究，比如我的GitHub名为thinkingOfBetty，所以我的仓库名字就是thinkingOfBetty.github.io。

然后按照下面的步骤进行操作：
（1）全局安装hexo npm install -g hexo
(2) 在空白文件下执行 hexo init 
(3) hexo g (生成)   hexo s（启动服务） 执行上面两个步骤之后访问localhost:4000端口可以访问到界面
（4）如果你想修改主题，可以访问hexo主题网站上对应的git地址，然后进行下载$类似的命令：git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia（下载好之后配置最外层的_config.yml中的theme）
（5）开始写文章：source中进行编写
（6）上传时要特别注意：master分支上只需要上传public中的内容，之后每次更新文件的时候只需要上传修改的对应静态文件。hexo分支上传的是环境文件。两个分支上都可以配置对应的.gitIgnore文件 
（7）下次再新的pc上进行开发时，可以先拉去master的内容，然后通过git checkout 本地分支名 origin/远程分支名
（8）在hexo分支上写文章，然后git add .;git commit -m xx;git push xx; hexo g -d ；之后复制对应的变更给到master中进行上传，此时即可看到界面发生改变。