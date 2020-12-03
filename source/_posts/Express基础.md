title: Express基础
date: 2015-07-08 22:31:10
tags: [Node.js,Express]
---

自己从Express官网API做的一些摘要。

安装express4.X `$ npm install -g express-generator`
	

<!--more-->

express.static(root, [options]);
`express.static`现在是express中唯一一个内置的中间件,用来为应用提供静态资源服务.

	//指定public为静态资源目录
	app.use(express.static(path.join(__dirname, 'public')));

## Application ##

app.set(name,value)：用来设置项目配置，常用的有:

	//views文件夹
	app.set('views', path.join(__dirname, 'views'));
	//视图渲染引擎
	app.set('view engine', 'jade');


**app.use([path,]function[,function...])**:大量的中间件可以放在`path`中，如果`path`没有指定的话，默认是`"/"`。

另外如果path的值是`"/"`时，这个路由会匹配所有路径，比如`"/user","/user/show","/user/show/info"`等等...因此express默认生成的项目中使用app.use()来捕获404异常.而且因为中间件的函数也都是写在path中，因此这些中间件的函数无论在哪个请求路径下都能执行。

	//将app.use写在路由规则的最后，当前面的路由无一能匹配时(传说中的404 NOT FOUND)，路由会经过app.use(function(){}),使用function()作为异常处理函数。
	app.use(function(req, res, next) {
	  var err = new Error('Not Found');
	  err.status = 404;
	  next(err);
	});

**中间件是顺序执行的**，因此include和use中间件的顺序很重要(可能存在依赖关系).

Express中路由的路径可以是字符串路径，路径模式，正则表达式或者数组。


	// GET /style.css etc
	app.use(express.static(__dirname + '/public'));
	//也可同时指定多个，请求资源时按顺序请求(最好给public优先权)
	app.use(express.static(__dirname + '/public'));
	app.use(express.static(__dirname + '/files'));
	app.use(express.static(__dirname + '/uploads'));

## Request ##
request对象代表HTTP请求，含有包括请求字符串，参数，请求体，请求头等等。**在Express4中，req.files默认在req中不可用**，因此如果想进行上传文件的功能，要使用例如[multer](https://www.npmjs.com/package/multer "multer")这样的中间件。

**req.body**:包含提交数据的键值对，默认是`undefined`，当你使用body-parsing中间件时它就被构建了。

**req.params**：一个包含映射路由参数的对象。例如，如果使用了路由`/user/:name`，则`name`属性`req.params.name`可用，该对象默认为`{}`.

**req.query**：包括query string(url中?后的部分)参数的对象，默认为空对象`{}`。
	

**req.cookie**:当使用`cookie-parser`时，它是一个含有cookies的对象，若req中不包含cookies，默认为空对象`{}`.

**req.get**：返回指定的请求头信息。

	req.get('Content-Type');
	// => "text/plain"

req.ip,req.hostname,req.protocol可以获取请求的ip,主机名,协议

## Response ##
response代表应用收到请求后返回的相应的HTTP相应。

res.download(path[,filename][,fn]):下载文件.

res.end():结束响应进程。

res.format(object):根据不同的请求头信息(也可以通过MIME TYPE)返回不同数据。

	res.format({
	  'text/plain': function(){
	    res.send('hey');
	  },
	
	  'text/html': function(){
	    res.send('<p>hey</p>');
	  },
	
	  'application/json': function(){
	    res.send({ message: 'hey' });
	  },
	
	  'default': function() {
	    // log the request and respond with 406
	    res.status(406).send('Not Acceptable');
	  }
	});

or

	res.format({
	  text: function(){
	    res.send('hey');
	  },
	
	  html: function(){
	    res.send('<p>hey</p>');
	  },
	
	  json: function(){
	    res.send({ message: 'hey' });
	  }
	});

**res.json**:返回JSON响应.
**res.redirect**:页面重定向.
**res.render(view[,locals][,callback])**:渲染视图.
**res.send([body])**.发送响应，`[body]`可以是`Buffer`，`Object`，`String`，`Array`.
res.set(field [, value]):设置响应头信息.

	res.set('Content-Type', 'text/plain');

	res.set({
	  'Content-Type': 'text/plain',
	  'Content-Length': '123',
	  'ETag': '12345'
	})

res.sendFile(path[,options][,fn]):

	app.get('/file/:name', function (req, res, next) {

	  var options = {
	    root: __dirname + '/public/',
	    dotfiles: 'deny',
	    headers: {
	        'x-timestamp': Date.now(),
	        'x-sent': true
	    }
	  };
	
	  var fileName = req.params.name;
	  res.sendFile(fileName, options, function (err) {
	    if (err) {
	      console.log(err);
	      res.status(err.status).end();
	    }
	    else {
	      console.log('Sent:', fileName);
	    }
	  });
	
	})
	
res.sendStatus: 设置响应HTTP状态码并且返回对应字符串信息。

res.sendStatus(200); // equivalent to res.status(200).send('OK');
res.sendStatus(404); // equivalent to res.status(404).send('Not Found');

	