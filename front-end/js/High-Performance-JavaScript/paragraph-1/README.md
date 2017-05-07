## 脚本位置
请记住，浏览器在解析到body标签之前，不会渲染页面的任何部分。把脚本放在页面顶部会导致明显的延迟，通常表现为显示空白页面，用户无法浏览内容，页无法页页面进行交互。  
即使现在很多浏览器支持并行下载Javascript文件，但是页面荏苒必须等待所有javascript代码下载并执行完成才能继续。  
所以应将所有script标签放在body标签地步

## 组织脚本

考虑到HTTP请求会带来额外的性能开销，减少页面中外链脚本文件的数量将会改善性能。（也不是绝对的将所有的文件合并成一个文件）

## 无阻塞脚本

无阻塞脚本的秘诀在于，在页面加载完成后才加载javascript代码。用专业术语来说，这意味着在window对象的load事件触发后再下载脚本。有一下几种方法现实这种效果：

###  延迟脚本
html4定义一个扩展属性：defer。以及H5引进的async属性，用于异步加载脚本。  
async与defer的相同点是采用并行下载，在下载过程中不会产生阻塞。区别在于执行时机，async是加载完成后**自动执行**，而defer需要**等待页面完成后执行**  
下面看个列子展示了defer属性如何影响脚本行为：
> H5规范定义：defer属性仅当src属性声明时才生效。
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <script defer>
    alert("defer")
  </script>
  <script>alert("script")</script>
  <script>
    window.onload = function(){
      alert('load')
    }
  </script>
</body>
</html>
```
在支持defer属性的浏览器傻姑娘，弹出的顺序是：script，defer，load。

### 动态脚本元素
这种技术的重点在于：无论何时启动下载，文件的下载和执行过成不会阻塞页面其它进程。  
如果页面其它脚本依赖该脚本时候，就会有问题。这种情况下，你必须跟踪并确保脚本下载完成且准备就绪。  

```javascript
var scrpt = document.createElement("script");
	script.type = 'text/javascript';
	//IE
	if( script.readyState){
		script.onreadystatechange = function(){
			if(script.readyState == 'loaded' || script.readyState == 'complete'){
				script.onreadstatechange = null;
				callback()
			}
		}
	}else{//firefox,Opera,chrome,safari
		script.onload = function(){
			callback()
		}
	}
	
	script.src = 'file.js';
	document.head.appendChild(script);
```
动态脚本加载凭借着它在跨浏览器兼容性和易用的优势，成为最通用的无阻塞加载解决方案。

### XMLHttpRequest 脚本注入

例：该方法的主要优点是，你可以下载JavaScript代码但不立即执行
```javascript
var xhr = new XMLHttpRequest();
	xhr.open('get','file.js',true)
	xhr.onreadystatechange = function(){
		if(xhr.readyState == 4){
			if(xhr.status >= 200 && xhr.status < 300 || xhr.status == 304){
				var script = document.createElement("script");
				script.type = 'text/javascript';
				script.text = xhr.responseText;
				document.body.appendChild(script);
			}
		}
	}
	xhr.send(null);
```
