layout: post
title: 使用nginx代理解决跨域问题
date: 2016-04-18 22:44:30
tags: ['nginx']
---

使用nginx代理解决跨域，方便快捷，只需几项配置，告别跨域烦恼~

<!--more-->

前端开发中跨域是最常见的问题之一，在开发过程中有一个情况: 我的项目代码在本地，API则在其他的域名下，这样在开发中测试功能就会受到跨域限制，想要修改本地代码，获取远程API接口数据测试，因此需要把API请求代理到remote端，项目代码本身则不需要代理。从前没尝试过这方面的研究，本来是把本地请求都代理到远程，发现访问的项目代码也变成远程的了，修改的功能得不到体现。于是需要一个更灵活的代理服务器来完成，nginx就可以做到。

Mac下安装nginx很简单，通过homebrew安装即可:


    brew install nginx

如果顺利的话，一条命令直接ok，不过也有不顺利的时候，比如在我今天安装时，install ok后，提示我`/some/path is not writable`,没有写权限，解决办法：`sudo chmod -R $(whoami) /some/path`,nginx安装完成，使用`sudo nginx`即可开启nginx服务。

接下来要对nginx进行配置，配置文件在`/usr/local/etc/nginx/nginx.conf`下，然后我们进行一点小修改~


    sudo vim /usr/local/etc/nginx/nginx.conf


我们主要修改`server`相关的配置，该字段包含虚拟主机的配置。


    server {
    
    	listen 80;
    	server localhost;
    	index index.html;
    	
    	root /path/to/your/project;
    	
    	location ^~/api/ {
    		proxy_pass http://api.com;
    	}
    
    }


`listen`和`server_name`分别指定了访问的域名和端口，比如如上配置会匹配`localhost`(默认80端口)，`root`指定了网站根目录的位置,`index`指定了首页文件名，可以写多个(html, htm, php...)，用空格隔开。

然后就是`location`，可以用来匹配你在该`server_name:port`下的访问路径，比如上面的`root`，还可以这么写：


    location / {
    	root /path/to/your/project;
    	...
    }


然后使用`proxy_pass`将API代理到你的API服务器上，就可以解决跨域问题了。

遇到的一个坑：
---
安装好nginx配置好后，访问localhost显示`403 forbidden`，在sof上找到了回答：当你的代理配置好后，如果index不存在，也会返回`403 forbidden`,不过我不是这个问题，而是另外一种：当用户需要获取(access)一个文件时，需要他的每一个父文件夹有x权限(可执行权限)才能获取到，这是一个很容易被忽略的问题。只要`chmod o+x /home`(任何拒绝访问的文件夹)即可。[详情](http://stackoverflow.com/questions/6795350/nginx-403-forbidden-for-all-files)这个问题看了好久，不过当时也是脑子犯浑了，看到forbidden就应该往权限上想的。。。

另外关于nginx的一些命令：


    //需要sudo下运行
    nginx //启动nginx服务
    nginx -s stop //退出
    nginx -s quit //退出
    nginx -s reload //重新加载配置文件重启


另外在写nginx配置文件的时候，一定要注意加`;`！！！而且在不断地改conf的过程中，还学会了一招：在vim中`:+<number>`可以直接跳`number`行。

目前涉及到的nginx用法就这么多，以后用到再进行补充。

End.




