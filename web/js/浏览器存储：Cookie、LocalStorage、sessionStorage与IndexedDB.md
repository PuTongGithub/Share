# 浏览器存储：Cookie、LocalStorage、sessionStorage与IndexedDB

有的时候，出于方便、缓存或是安全的考虑，我们会将一些数据存储在前端浏览器上，比如类似这样的功能：记录用户的历史输入，用于搜索框的自动补全。在面对类似的功能需求时，我们就会需要到浏览器存储功能的帮助。本篇文章将梳理对比四种常用的浏览器存储功能。

### 1. Cookie

首先是我们经常会遇到的Cookie，不过**Cookie 的本职工作并非本地存储，而是“维持状态”**。由于HTTP协议是无状态的，于是浏览器在HTTP请求报文中维护了一个参数Cookie，主要是用来记录并保持浏览器与服务器之间的会话状态。也就是说，浏览器放置在Cookie中的内容，服务端也是可以在接收HTTP请求的时候获取到的。

#### 如何使用Cookie

Cookie本身的数据内容是以键值对的方式存储的，通过JavaScript里的`document.cookie`便可以实现对Cookie的操作：

```javascript
// 设置cookie，这条语句将会覆盖掉旧的cookie值
document.cookie = "username=Bill Gates";
// 设置cookie并指定有效期（不指定时本条cookie将在浏览器关闭时删除）
document.cookie = "username=John Doe; expires=Sun, 31 Dec 2017 12:00:00 UTC";
// 设置cookie并指定生效路径（不指定时该cookie仅在当前页有效）
document.cookie = "username=Bill Gates; expires=Sun, 31 Dec 2017 12:00:00 UTC; path=/";
// 读取cookie，参数x将会获取到所有cookie的值
var x = document.cookie;
// 删除cookie，通常采用将有效期指定为过去的方式来删除一个cookie
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
```

### 2. LocalStorage

不同于Cookie，LocalStorage用于保存长期数据，下一次访问该网站的时候，网页可以直接读取以前保存的数据，这些保存的数据不会参与服务端通信。

#### 如何使用LocalStorage

LocalStorage也是以键值对的方式进行存储的，但是它的封装比Cookie要好用一些，在JavaScript中通过`window.localStorage`对象就可以来使用它了：

```javascript
// 获取LocalStorage对象
var localStorage = window.localStorage;
// 存储数据
localStorage.setItem("key", "value");
// 读取数据
var value = localStorage.getItem("key");
// 删除数据
localStorage.removeItem("key");
```

### 3. SessionStorage

SessionStorage与LocalStorage相似，区别在于SessionStorage存储的数据仅存在于本次会话中。

#### 如何使用SessionStorage

与LocalStorage类似：

```javascript
// 获取SessionStorage对象
var sessionStorage = window.sessionStorage;
// 存储数据
sessionStorage.setItem("key", "value");
// 读取数据
var value = sessionStorage.getItem("key");
// 删除数据
sessionStorage.removeItem("key");
```

### 4. IndexedDB

IndexedDB是一个运行在浏览器上的非关系型数据库，用于存储客户端的大量结构化数据(包括文件和blobs)，一般来说存储空间不会小于250M。

#### 如何使用IndexedDB

IndexedDB使用了索引来存储键值对数据，不同的是，除了字符串，它还可以存储二进制数据；另外，IndexedDB使用了异步的方式来完成操作，首先，我们需要获取数据库操作对象：

```javascript
// 使用open方法打开一个数据库的连接，获取数据库连接对象，第一个参数是数据库的名称，第二个参数是数据库的版本（大于0）
var openRequest = window.indexedDB.open("test",1);
// 数据库操作对象
var db；

// 我们可以在这个对象上注册4个回调函数
// 第一次打开该数据库，或者数据库版本发生变化
openRequest.onupgradeneeded = function(e) {
    console.log("Upgrading...");
}
// 上一次的数据库连接还未关闭
openRequest.blocked = function(e) {
    console.log("Blocked...");
}
// 打开成功，获取到数据库操作对象
openRequest.onsuccess = function(e) {
    console.log("Success!");
    db = e.target.result;
}
// 打开失败
openRequest.onerror = function(e) {
    console.log("Error");
    console.dir(e);
}
```

获取到数据库操作对象后我们就可以利用它来操作IndexedDB了，详情请参考[JavaScript IndexedDB：浏览器端数据库](https://www.w3cschool.cn/javascript_guide/javascript_guide-rcfy26a4.html)。

### 四种存储的特性对比

| 特性         | Cookie         | LocalStorage                         | SessionStorage                                       | IndexedDB                |
| ------------ | -------------- | ------------------------------------ | ---------------------------------------------------- | ------------------------ |
| 数据生命周期 | 手动指定       | 一直存在                             | 一次会话                                             | 一直存在                 |
| 数据存储大小 | 单条KV可存储4K | 2.5M~10M之间，取决于浏览器实现       | 同LocalStorage                                       | 一般情况下至少有250M     |
| 与服务端通信 | 参与           | 不参与                               | 不参与                                               | 不参与                   |
| 存储数据类型 | 字符串         | 字符串                               | 字符串                                               | 字符串、二进制           |
| 数据获取操作 | 同步           | 同步                                 | 同步                                                 | 异步（支持事务）         |
| 同源限制     | 手动指定       | 有（作用于相同协议、主机名、端口下） | 有（与LocalStorage类似，但额外要求作用于同一窗口下） | 有（无法跨域访问数据库） |



###### 参考内容链接

1. [深入了解浏览器存储：对比Cookie、Local/sessionStorage与IndexedDB](https://zhuanlan.zhihu.com/p/61704951)
2. [JavaScript Cookies](https://www.w3school.com.cn/js/js_cookies.asp)
3. [Window localStorage 属性](https://www.runoob.com/jsref/prop-win-localstorage.html)
4. [Window sessionStorage 属性](https://www.runoob.com/jsref/prop-win-sessionstorage.html)
5. [JavaScript IndexedDB：浏览器端数据库](https://www.w3cschool.cn/javascript_guide/javascript_guide-rcfy26a4.html)
6. [IndexedDB：浏览器端数据库](https://wohugb.gitbooks.io/javascript/bom/indexeddb.html)

