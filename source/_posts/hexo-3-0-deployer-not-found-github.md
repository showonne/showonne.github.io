title: 'hexo 3.0 deployer not found:github'
date: 2015-06-17 00:11:42
tags: Hexo
---

hexo升级到3.0了，刚升级完，发现deploy出错:`error deployer not found:github`，原因是更新到3.0之后，config.yml里deploy的type需要改为`git(原本是github)`,另外要先安装一下`hexo-deployer-git `.

`npm install hexo-deployer-git --save`


