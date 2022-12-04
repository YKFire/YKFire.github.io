---
title: HTML5 Web存储
date: 2022-12-04 11:13:00 +0800
categories: [技术科普]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# HTML5 Web存储

## 一、简介

​	HTML5 Web 存储（webStorage）是本地存储，存储在客户端，包括**localStorage和SessionStorage**。HTML5 Web 存储是**以键/值对的形式**存储的，通常以**字符串存储**

## 二、localStorage

​	localStorage生命周期是永久，除非主动清除localStorage信息，否则这些信息将永远存在。存放数据大小为一般为5MB,而且它仅在客户端（即浏览器）中保存，不参与和服务器的通信

```javascript
使用方法:

// 1、保存数据到本地
// 第一个参数是保存的变量名，第二个是赋给变量的值
localStorage.setItem('Author', 'local'); 
 
// 2、从本地存储获取数据
localStorage.getItem('Author');
 
// 3、从本地存储删除某个已保存的数据
localStorage.removeItem('Author');
 
// 4、清除所有保存的数据
localStorage.clear();
```

![localStorage](/assets/blog_res/2022-12-04-WebStorage.assets/image-20221204115357771.png)

## 三、sessionStorage

​	sessionStorage 属性允许你访问一个对应当前源的 session Storage 对象。它与 localStorage 相似，不同之处在于 **localStorage 里面存储的数据没有过期时间设置**，而存储在 sessionStorage 里面的数据**在页面会话结束时会被清除**

- 页面会话在浏览器打开期间一直保持，并且重新加载或恢复页面仍会保持原来的页面会话
- 在新标签或窗口打开一个页面时会复制顶级浏览会话的上下文作为新会话的上下文，这点和 session cookies 的运行方式不同
- 打开多个相同的URL的Tabs页面，会创建各自的sessionStorage
- 关闭对应浏览器窗口（Window）/ tab，会清除对应的sessionStorage

```javascript
使用方法:

// 1、保存数据到本地
// 第一个参数是保存的变量名，第二个是赋给变量的值
sessionStorage.setItem('Author', 'YKFire');
 
// 2、从本地存储获取数据
sessionStorage.getItem('Author');
 
// 3、从本地存储删除某个已保存的数据
sessionStorage.removeItem('Author');
 
// 4、清除所有保存的数据
sessionStorage.clear();
```

![sessionStorage](/assets/blog_res/2022-12-04-WebStorage.assets/image-20221204115558776.png)

## 四、复杂数据存储

> 以上示例展示的都是简单数据的存储，当要存储的数据是一个对象或数组的时候，直接存储是不行的

错误代码：

```javascript
var user = {
    username: 'YKFire',
    password: '123456'
 
};
sessionStorage.setItem('user', user);
console.log(sessionStorage.getItem('user'));
```

![对象信息无法正确展示](/assets/blog_res/2022-12-04-WebStorage.assets/image-20221204120040800.png)

因此我们在存取数据的时候需要转换格式：

- **存储数据前：利用JSON.stringify将对象转换成字符串**
- **获取数据后：利用JSON.parse将字符串转换成对象**

正确代码：

```javascript
var user = {
    username: 'YKFire',
    password: '123456'
};
user = JSON.stringify(user);
sessionStorage.setItem('user', user);
 
var account = sessionStorage.getItem('user');
console.log(account);
 
account = JSON.parse(account)
console.log(account);
```

![对象信息正确展示](/assets/blog_res/2022-12-04-WebStorage.assets/image-20221204121024115.png)

## 五、跨页面传值的方式

> 此部分讲解的是**客户端中页面与页面**的传值方式，切忌与**客户端与服务端之间的传值方式**搞混

### 1、通过url传值

```javascript
上一个页面发送：
location.href="跨页面1-2.html?age=18&gender=man";

下一个页面接受：
//1、location.search获取get请求的参数   获取到的数据，是以?开头的
var search=location.search;
//2、如果还想要获取确定的数据，可以解析字符串
function parse(search){
    //从第二个字符开始截取   ，获取到第二个开始后面所有的字符
    var str=search.substring(1);
    var result={};
    //分割字符串  -->产生字符串数组
    var strs=str.split("&");
    //遍历数组中的每一个元素
    strs.forEach(function(v){
        //伪代码：v="age=18"
        var keyvalue=v.split("=");
        var name=keyvalue[0];
        var value=keyvalue[1];
        result[name]=value;
    })
    return result;
}
 
var r=parse(search);
```

### 2、使用WebStorage传值

> localStroage和sessionStorage使用大致相同，他们的不同之处在于，localstroage是永久保存，而sessionstroage是会话存在，当会话结束，sessionstroage保存值也会清空

```javascript
存储对象的正确的方式：

var user={name:"YKFire",age:16};
var user1=JSON.stringify(user1);      //将对象"序列化"为JSON数据(字符串格式)
localStorage.setItem("user",user1);  //以字符串格式存储信息
var a=localStorage.getItem("user");    //获取存储的信息，也是字符串格式
var b=JSON.parse(a);      //将JSON数据反序列化为对象
```

### 3、使用cookie保存

```javascript
//1、保存一条数据
document.cookie="name=abc";
document.cookie="age=18";
//2、获取所有数据
var cookie=document.cookie;
console.log(cookie);  //"name=abc; age=18; PHPSESSID=fr1njdv6apf3neoj5nehntrps7"
//之后可以解析字符串，获取指定的数据内容
//3、设置cookie的有效期
document.cookie="id=666;expires="+new Date("2017-10-22 08:00");
```

## 六、cookie存储与Web存储的区别与应用场景

### 1、cookie与session

#### 1.简介

​	cookie和session都是用来**跟踪浏览器用户身份**的会话方式

![cookie与session](/assets/blog_res/2022-12-04-WebStorage.assets/image-20221204122759138.png)

#### 2.区别

##### 保持状态

​	cookie保存在浏览器端，session保存在服务器端

##### 使用方式

 cookie机制：

> ​	**如果不在浏览器中设置过期时间，cookie被保存在内存中**，生命周期随浏览器的关闭而结束，这种cookie简称会话cookie。**如果在浏览器中设置了cookie的过期时间，cookie被保存在硬盘中**，关闭浏览器后，cookie数据仍然存在，直到过期时间结束才消失
>
> ​	Cookie是服务器发给客户端的特殊信息，cookie是以文本的方式保存在客户端，每次请求时都带上它

session机制：

> ​	当服务器收到请求需要创建session对象时，首先会检查客户端请求中是否包含sessionid。**如果有sessionid，服务器将根据该id返回对应session对象**。如果客户端请求中没有sessionid，服务器会创建新的session对象，并把sessionid在本次响应中返回给客户端。通常使用cookie方式存储sessionid到客户端，**在交互中浏览器按照规则将sessionid发送给服务器**。**如果用户禁用cookie，则要使用URL重写**，可以通过response.encodeURL(url) 进行实现；API对encodeURL的结束为，当浏览器支持Cookie时，url不做任何处理；当浏览器不支持Cookie的时候，将会重写URL将SessionID拼接到访问地址后

##### 存储内容

​	cookie只能保存字符串类型，以文本的方式；session通过类似与Hashtable的数据结构来保存，能支持任何类型的对象(session中可含有多个对象)

##### 存储大小

​	单个cookie保存的数据不能超过4kb；session大小没有限制

##### 安全性

​	session的安全性大于cookie，针对cookie所存在的攻击：Cookie欺骗，Cookie截获

原因：

- sessionID存储在cookie中，若要攻破session首先要攻破cookie
- sessionID是要有人登录，或者启动session_start才会有，所以攻破cookie也不一定能得到sessionID
- 第二次启动session_start后，前一次的sessionID就是失效了，session过期后，sessionID也随之失效
- sessionID是加密的

#### 3.应用场景

**cookie：**

- 判断用户是否登陆过网站，以便下次登录时能够实现自动登录（或者记住密码）。如果我们删除cookie，则每次登录必须从新填写登录的相关信息
- 保存上次登录的时间等信息
- 保存上次查看的页面
- 浏览计数

**session：**

- Session用于保存每个用户的专用信息，变量的值保存在服务器端，通过SessionID来区分不同的客户
- 网上商城中的购物车
- 将某些数据放入session中，供同一用户的不同页面使用

#### 4.缺点

**cookie：**

- 大小受限
- 用户可以操作（禁用）cookie，使功能受限
- 安全性较低
- 有些状态不可能保存在客户端
- 每次访问都要传送cookie给服务器，浪费带宽
- cookie数据有路径（path）的概念，可以限制cookie只属于某个路径下

**session：**

- Session保存的东西越多，就越占用服务器内存，对于用户在线人数较多的网站，服务器的内存压力会比较大
- 依赖于cookie（sessionID保存在cookie），如果禁用cookie，则要使用URL重写，不安全
- 创建Session变量有很大的随意性，可随时调用，不需要开发者做精确地处理，所以，过度使用session变量将会导致代码不可读而且不好维护

### 2、localStorage与sessionStorage

#### 1.简介

> WebStorage的目的是克服由cookie所带来的一些限制，当数据需要被严格控制在客户端时，不需要持续的将数据发回服务器。
>
> WebStorage两个主要目标：（1）提供一种在cookie之外存储会话数据的路径。（2）提供一种存储大量可以跨会话存在的数据的机制。
>
> HTML5的WebStorage提供了两种API：localStorage（本地存储）和sessionStorage（会话存储）

#### 2.区别

##### 生命周期

localStorage:

> 其生命周期是永久的，关闭页面或浏览器之后localStorage中的数据也不会消失。localStorage除非主动删除数据，否则数据永远不会消失。

sessionStorage：

> 其生命周期是在仅在当前会话下有效；sessionStorage引入了一个“浏览器窗口”的概念，sessionStorage是在同源的窗口中始终存在的数据。只要这个浏览器窗口没有关闭，即使刷新页面或者进入同源另一个页面，数据依然存在。但是sessionStorage在关闭了浏览器窗口后就会被销毁。同时独立的打开同一个窗口同一个页面，sessionStorage也是不一样的。

##### 存储大小

​	localStorage和sessionStorage的存储数据大小一般都是：5MB

##### 存储位置

​	localStorage和sessionStorage都保存在客户端，不与服务器进行交互通信

##### 存储内容类型

​	localStorage和sessionStorage**只能存储字符串类型**，对于复杂的对象可以使用ECMAScript提供的JSON对象的stringify和parse来处理

##### 应用场景

localStoragese：

​	常用于长期登录（+判断用户是否已登录），适合长期保存在本地的数据

sessionStorage：

​	敏感账号一次性登录

#### 3.优点

- 存储空间更大：cookie为4KB，而WebStorage是5MB
- 省网络流量：WebStorage不会传送到服务器，存储在本地的数据可以直接获取，也不会像cookie一样每次请求都会传送到服务器，所以减少了客户端和服务器端的交互，节省了网络流量
- 对于那种只需要在用户浏览一组页面期间保存而关闭浏览器后就可以丢弃的数据，sessionStorage会非常方便
- 快速显示：获取数据时可以从本地获取会比从服务器端获取快得多，所以速度更快
- 安全性：WebStorage不会随着HTTP header发送到服务器端，所以安全性相对于cookie来说比较高一些，不会担心截获，但是仍然存在伪造问题
- WebStorage提供的数据操作方法比cookie方便