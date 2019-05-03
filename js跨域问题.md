#### 什么是跨域

我们经常会看到上边类似的报错，Access-Cotrol-Allow-Origin 这是典型的跨域报错。其实我们通常所说的跨域是狭义的，是由浏览器同源策略限制的一类请求场景。那什么是同源策略呢？


#### 什么是同源策略

```
	浏览器安全的基石是“同源策略”（same-origin policy）。Netscape 公司引入浏览器，目前，所有浏览器都实行这个政策。
	
	所谓同源是指“协议+域名+端口”三者相同，即便两个不同的域名指向同一个ip地址，也非同源。
	
	它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS,CSFR等攻击。
```

所谓“同源”指的是“三个相同”。

```
	协议相同域名相同端口相同
```

举例来说：http://www.a.com:3000/index.html这个网址，协议是http://，域名是www.a.com，端口是3000（我们经常看的网址没有，是因为默认端口80可以省略）。

### 常见的跨域场景

```
	URL                                      说明                    是否允许通信
http://www.a.com/a.js
http://www.a.com/b.js         同一域名，不同文件或路径           允许,属于同源
http://www.a.com/lab/c.js

http://www.a.com:8000/a.js
http://www.a.com/b.js         同一域名，不同端口                不允许
 
http://www.a.com/a.js
https://www.a.com/b.js        同一域名，不同协议                不允许
 
http://www.a.com/a.js
http://192.168.4.12/b.js      域名和域名对应相同ip              不允许
 
http://www.a.com/a.js
http://x.a.com/b.js           主域相同，子域不同                不允许
http://a.com/c.js
 
http://www.a.com/a.js
http://www.a.com/b.js         不同域名                         不允许

```


#### 同源策略限制以下几种行为：

```
（1） Cookie、LocalStorage 和 IndexDB 无法读取。（2） DOM 无法获得。（3） AJAX 请求不能发送。
```

#### 跨域解决方案

```
	1、 通过jsonp跨域
	2、CORS
	3、 document.domain + iframe跨域
	4、 location.hash + iframe
	5、 window.name + iframe跨域
	6、 postMessage跨域
	7、 nginx代理跨域
	8、 nodejs中间件代理跨域
	9、 WebSocket协议跨域
```

##### 1.通过jsonp跨域

> 通常为了减轻web服务器的负载，我们把js、css、img等静态资源分离到另一台独立域名的服务器上，在html页面中在通过相应的标签从不同域名下加载静态资源，而被浏览器允许，基于此原理，我们可以通过动态创建script，再请求一个带参网址实现跨域通信。jsonp正是利用这个特性来实现的。


###### 优缺点：

 1. JSONP是服务器与客户端跨源通信的常用方法，最大特点就是简单适用，老式浏览器全部支持，服务器改造非常小。

 2. 只能实现get一种请求，不安全，容易遭到xss攻击。

##### CORS

> CORS是一个W3C标准，全称是“跨域资源共享”（Cross-origin resource sharing）它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制
> 

ps: 普通跨域请求，只服务端设置Access-Control-Allow-Origin即可，前端无需设置，若要带cookie请求：前后端都需要设置，由于同源策略的限制，所读取的cookie为跨域请求接口所在域的cookie，而非当前页。


#### 优缺点：

1. 目前，所有浏览器都支持该功能（IE8+ : IE8/9 需要设置使用XDomainRequest对象来支持CORS）,CORS也已经成为主流的跨域解决方案。
2. 整个CORS通信过程，都是浏览器自动完成，不需要用户参与，对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样，浏览器一旦发现AJAX请求跨域，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会感觉。
3. CORS和JSONP的使用目的相同，但是比JSONP更强大，JSONP只支持GET请求，CORS支持所有类型的HTTP请求，JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据

### 两种请求

> 浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）

只要同时满足以下两大条件，就属于简单请求，凡是不同时满足下面两个条件，就属于非简单请求。

```
	1. 请求方法是以下三种方法之一：HEAD、GET、POST
	2. HTTP的头信息不超出以下几种字段：AcceptAccept-LanguageContent-LanguageLast-Event-IDContent-Type：只限于三个值
	（1）application/x-www-form-urlencoded、
	（2）multipart/form-data、
	（3）text/plain

```

##### 简单请求：

简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息中，增加一个Origin字段，

##### 非简单请求：

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为”预检“请求（preflight）

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段，只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。



+ Access-Control-Allow-Origin

该字段是必须的，它的值要么是请求是**Origin**字段的值，要么是一个**\***,表示接受任意域名的请求。

+ Access-Control-Allow-Credentials

改字段可选，它的值是一个布尔值，表示是否允许发送Cookie,默认情况下，Cookie不包括在CORS请求中，设为true,即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器，这个值也只能设为true，如果服务器不要浏览器发送Cookie,删除该字段即可。

+ Access-Control-Expose-Headers

该字段可选，CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。


##### withCredentials 属性

CORS 请求默认不发送Cookie和HTTP认证信息,如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段，

```
	Access-Control-Allow-Credentials: true
```

开发者必须在AJAX请求中打开withCredentials属性

```
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```

否则，即使服务器同意发送Cookie,浏览器也不会发送，或者，服务器要求设置Cookie,浏览器也不会处理。

但是，如果省略withCredentials设置，有的浏览器还是会一起发送Cookie,这时，可以显式关闭withCredentials。

```
	xhr.withCredentials = false;
```

需要注意的是，如果要发送Cookie, Access-Control-Allow-Origin 就不能设置为星号，必须指定明确的、与请求网页一致的域名，同时，Cookie必然遵循同源政策，只能用服务器域名设置的Cookie才会上传，其它域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie.

##### Access-Control-Request-Method 

该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。

##### Access-Control-Request-Headers

该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段

```
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

##### Access-Control-Allow-Methods

该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

##### Access-Control-Allow-Headers

如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

##### Access-Control-Allow-Credentials

该字段与简单请求的含义相同

##### Access-Control-Max-Age

该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

##### document.domain + iframe跨域

```
	此方案仅限主域相同，子域不同的跨域应用场景（网页一级域名相同，只是二级域名不同）实现原理：两个页面都通过js强制设置document.domain 为基础主域，就实现了同域
```

##### location.hash

```
	实现原理：a与b跨域相互通信，通过中间页c来实现（且c与a是同域），三个页面，不同域之间利用iframe的location.hash传值，相同域之间直接js访问来通信。
	
	具体实现：A域： a.html -> B域：b.html -> A域：c.html, a与b不同域只能通过hash值单向通行，b与c也不同域也只能单向通信。但c与a同域，所以c可通过parent.parent访问a页面所有对象。
```

##### window.name + iframe跨域

浏览器窗口有window.name 属性，这个属性最大的特点是：无论是否同源，只要是同一个窗口内，前一个网页设置了这个属性，后一个网页可以读取它，并且支持非常长的name值（2MB）;

父窗口先打开一个子窗口，载入一个不同源的网页，改网页将信息写入window.name 属性。

接着，子窗口调回一个与主窗口同域的网址。
```
	location = "http://parent.url.com/xxx.html";
```

然后，主窗口就可以读取子窗口的window.name了。

```
	var data = document.getElementById('myFrame').contentWindow.name;

```

这种方法的优点是，window.name容量很大，可以放置非常长的字符串；缺点是必须监听子窗口window.name属性的变化，影响网页性能。


##### ngnix 代理跨域

1. nanix 配置解决iconfont 跨域

浏览器跨域访问js、css、img等常规静态资源被同源策略许可，但iconfont字体文件(eot|otf|ttf|woff|svg)例外，此时可在nginx的静态资源服务器中加入以下配置。

```
location / {
  add_header Access-Control-Allow-Origin *;
}
```

#####  nginx反向代理接口跨域

跨域原理： 同源策略是浏览器的安全策略，不是HTTP协议的一部分。服务器端调用HTTP接口只是使用HTTP协议，不会执行JS脚本，不需要同源策略，也就不存在跨越问题。

实现思路：通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。




```

```

##### 小程序开发中常用的一些基础组件

+ 网络请求封装

+ 一些常用的正则校验工具类

+ Page创建的封装，便于一些全局配置

+ Toast弹窗



+ 1、商品详情、下单代码逻辑优化
+ 2、工程一些不需要的资源清理，
+ 3、工程代码一些布局警告问题优化
+ 4、工程中一些兼容性组件替换。
+ 5、