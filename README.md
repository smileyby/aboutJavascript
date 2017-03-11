Javascript 的装载和执行
=====================

通常来说，浏览器对于Javascript的运行有两大特性：

1.	载入后马上执行
2.	执行时会阻塞页面后续的内容（包括页面的渲染/其他资源的加载）。于是，如果有多个js文件被引入，那么对于浏览器来说，这些js文件被穿行地载入，并一次执行。

因为javascrit可能会操作HTML文档的DOM树，所以，浏览器一般都不会像并行下载css文件并行下载js文件，因为这是js文件的特殊性造成的。所以，如果你的javascript像操作后面的DOM元素，基本上来说，浏览器都会报错说对象找不到。因为javascript执行时，后面的html被阻塞住了，DOM树时还没有后面的DOM节点。随意程序也就报错了。

## 传统的方式

所以，当你在代码中写下如下代码：

```javascript
	<script type="text/javascript" src="http://coolshell.cn/asyncjs/alert.js"></script>
```

基本上来说，head例的上的<script>标签会阻塞后续资源的载入以及整个页面的生成。

所以，你知道为啥那么有很对网站把javascript放在网页的最后面了，要么就是动用window.onload或是document ready之类的事件。

另外，因为绝大多数哦的javascrit代码并不需要等页面，即：异步载入。那我们如何异步载入呢？

## document.write 方式

于是，你可能以为document.write()这种方式能够解决阻塞问题。你当然会觉得，document.write了的<script>标签后就可以执行后面的东西了，者没错。对于在同一个script标签例的javascript的代码来说，是这样的，但是对于整个页面来说，这个还是会阻塞，下面时一段测试代码：

```javascript

	<script type="text/javascript" language="javascript">
	    function loadjs(script_filename) {
	        document.write('<' + 'script language="javascript" type="text/javascript"');
	        document.write(' src="' + script_filename + '">');
	        document.write('<'+'/script'+'>');
	        alert("loadjs() exit...");
	    }
	 
	    var script = 'http://coolshell.cn/asyncjs/alert.js';
	 
	    loadjs(script);
	    alert("loadjs() finished!");
	</script>
	 
	<script type="text/javascript" language="javascript">
	   alert("another block");
	</script>

```

[测试页面](http://coolshell.cn/asyncjs/async_test02.html)

## script的defer和async属性

IE自从IE6就支持defer，如：

```javascript
	<script defer type="text/javascript" src="./alert.js" ></script>
```

对于IE来说，这个标签会让IE并行下载js文件，并且把其执行hold到了整个DOM装载完毕（DOMContentLoaded），多个defer的<script>在窒息感时也会按照其出现的顺序来运行。但是因为这个defer只时IE专用，所以一般用的比较少。

而我们标砖的HTML5也加入了一个异步载入javascript的属性：async，无论你对它都什么样的值，只要它出现，它就开始异步加载js文件。但是，async的异步加载会有一个严重的问题，那就是它忠实地碱性者“载入后马上执行”这条军规，所以，虽然它并不阻塞页面渲染，但是你也无法控制他执行的次序和时机。

支持async标签的浏览器是：Firefox3.6+，Chrome8.0+，Safari5.0，IE10+。Opera还不支持[参考这里](http://caniuse.com/#feat=script-async)所以这个方法也不是太好。因此不是所有的浏览器你都能行。

## 动态创建DOM方式

这种方式可能是用的最多的了。

```javascript

	function loadjs(script_name){
		var script = doucment.createElement('script');
		script.setAttribute('type', 'text/javascript');
		script.setAttribute('src', script_name);
		script.setAttribute('id', 'my_script_id');
	
		script_id = document.getElementById('my_script_id');
		if(script_id){
			document.getElementsByTagName('head')[0].removeChild(script_id);
		}
		document.getElementsByTagName('head')[0].appendChild(script);	
	};

	var script = 'http://coolshell.cn/asyncjs/alert.js';
	loadjs(script);
	
```

这个方式几乎成了标准的异步载入js文件的方式。

## 按需异步载入js

上面的那个DOM方式的例子解决了异步载入javascript的问题，但是没有解决我们想让它按照我们指定的时机运行的问题。所以，我们只需要把上面的那个DOM方式帮到某个事件上来就可以了。

比如：

绑定在window.load事件上-[示例](http://coolshell.cn/asyncjs/async_test04.html)

绑定在特定的事件上-[示例](http://coolshell.cn/asyncjs/async_test05.html)

## 更多

但是，绑定在某个特定事件上这个事似乎又过了点，因此只有在点击的时候才会去真正的下载js，这又会太慢了。，我们想要异步地把嘉实稳键下载到本地，但又不执行，仅当在我们想要执行的时候去执行。

要是我们又下面的方式就好了：

```javascript

	var script = document.createElement("script");
	script.noexecute = true;
	script.src = "alert.js";
	document.body.appendChild(script);
	 
	//后面我们可以这么干
	script.execute();

```

可惜的是，这只是一个美丽的梦境，今天我们的javascript还比较原始，这个js梦还没有实现。

所以，我们的程序员只能是哟个hack的方式来搞。

有的程序员使用非标准的script的type来实现。如：

```javascript
	<script type=cache/script src="./alert.js"></script>
```

因为“cache/script”,这个东西根本就不能被浏览器歇息，所以浏览器也就不能把alert.js当javascript执行，但是它又要去下载嘉实稳键，所以就可以搞定了。可惜的是，webkit严格服从了HTML标准--对于这个不认识的东西，直接删除，什么也不干，于是我们的梦又破了。

所以，我们需要在hack以下，就是preload图片那样，我们可以动用object标签（也可以使用iframe标签），于是我们有了下面这样的代码：

```javascript
	function cachejs(script_filename){
	    var cache = document.createElement('object');
	    cache.data = script_filename;
	    cache.id = "coolshell_script_cache_id";
	    cache.width = 0;
	    cache.height = 0;
	    document.body.appendChild(cache);
	}
```

[示例](http://coolshell.cn/asyncjs/async_test06.html)

在Chrome下按Ctrl+Shift+I。切换到network页，你可以看到下载了alert.js但是没有执行，因为浏览器缓存了，不会再从服务器上下载alert.js，所以，就能保证执行速度了。

最后在提两个js，一个事[ControlJs](http://stevesouders.com/controljs/),另一个是[HeadJs](http://headjs.com/)都是专门用来做异步load javascript文件的。







