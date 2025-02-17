本文资源感谢   廖雪峰老师  https://www.liaoxuefeng.com/wiki/1022910821149312/1023025235359040

# 一.基本模块

### 0.1 global

我们都知道，JavaScript有且仅有一个全局对象，在浏览器中，叫`window`对象。而在Node.js环境中，也有唯一的全局对象，但不叫`window`，而叫`global`，这个对象的属性和方法也和浏览器环境的`window`不同。

- console，我们在初体验时，使用了console，它可不是浏览器中的console对象，使用的是node中的console

- process，和进程相关的对象

- setInterval，同理，也是node中的，不是浏览器中的

- **require()**，它是全局对象global中的一个方法，用于在js文件中引入另外的文件
    - require() 方法可以在js文件中加载另外的js文件（模块）
    - require() 方法可以在js文件中加载json文件
    
- __dirname，当前执行文件的绝对路径（在js文件中使用）

- __filename，当前执行文件的绝对路径，包含文件名（在js文件中使用）

> 上述：__dirname、\_\_filename、exports、module、require 这5个不能通过global调用，但是可以直接使用

### 0.2 process

`process`也是Node.js提供的一个对象，它代表当前Node.js进程。通过`process`对象可以拿到许多有用信息：

```js
> process === global.process;
true
> process.version;
'v5.2.0'
> process.platform;
'darwin'
> process.arch;
'x64'
> process.cwd(); //返回当前工作目录
'/Users/michael'
> process.chdir('/private/tmp'); // 切换当前工作目录
undefined
> process.cwd();
'/private/tmp'
```

JavaScript程序是由事件驱动执行的单线程模型，Node.js也不例外。Node.js不断执行响应事件的JavaScript函数，直到没有任何响应事件的函数可以执行时，Node.js就退出了。

如果我们想要在下一次事件响应中执行代码，可以调用`process.nextTick()`：

用Node执行上面的代码`node test.js`，你会看到，打印输出是：

```js
nextTick was set!
nextTick callback!
```

这说明传入`process.nextTick()`的函数不是立刻执行，而是要等到下一次事件循环。

Node.js进程本身的事件就由`process`对象来处理。如果我们响应`exit`事件，就可以在程序即将退出时执行某个回调函数：

```js
// 程序即将退出时的回调函数:
process.on('exit', function (code) {
    console.log('about to exit with code: ' + code);
});
```

### 0.3 判断JavaScript执行环境

有很多JavaScript代码既能在浏览器中执行，也能在Node环境执行，但有些时候，程序本身需要判断自己到底是在什么环境下执行的，常用的方式就是根据浏览器和Node环境提供的全局变量名称来判断：

```javascript
if (typeof(window) === 'undefined') {
    console.log('node.js');
} else {
    console.log('browser');
}
```

## 1.1 fs模块

### 1.1.1 读文件

#### ① 异步读文件

按照JavaScript的标准，异步读取一个文本文件的代码如下：

```js
var fs = require('fs');

fs.readFile('./123.html', 'utf-8', function (err, data) {
    if (err) console.log(err);
    console.log(data);
});
```

请注意，`sample.txt`文件必须在当前目录下，且文件编码为`utf-8`。

异步读取时，传入的回调函数接收两个参数，当正常读取时，`err`参数为`null`，`data`参数为读取到的String。当读取发生错误时，`err`参数代表一个错误对象，`data`为`undefined`。这也是Node.js标准的回调函数：第一个参数代表错误信息，第二个参数代表结果。后面我们还会经常编写这种回调函数。

由于`err`是否为`null`就是判断是否出错的标志，所以通常的判断逻辑总是：

```js
if (err) {
    // 出错了
} else {
    // 正常
}
```

如果我们要读取的文件不是文本文件，而是二进制文件，怎么办？

下面的例子演示了如何读取一个图片文件：

```js
var fs = require('fs');

fs.readFile('./timg.jpg', function (err, data) {
    if (err) {
        console.log(err);
    } else {
        console.log(data);
        console.log(data.length + ' bytes');
    }
});
```

当读取二进制文件时，不传入文件编码时，回调函数的`data`参数将返回一个`Buffer`对象。在Node.js中，`Buffer`对象就是一个包含零个或任意个字节的数组（注意和Array不同）。

`Buffer`对象可以和String作转换，例如，把一个`Buffer`对象转换成String：

```js
// Buffer -> String
var text = data.toString('utf-8');
console.log(text);
```

或者把一个String转换成`Buffer`：

```js
// String -> Buffer
var buf = Buffer.from(text, 'utf-8');
console.log(buf);
```

#### ② 同步读文件

除了标准的异步读取模式外，`fs`也提供相应的同步读取函数。同步读取的函数和异步函数相比，多了一个`Sync`后缀，并且不接收回调函数，函数直接返回结果。

用`fs`模块同步读取一个文本文件的代码如下：

```js
var fs = require('fs');

var data = fs.readFileSync('sample.txt', 'utf-8');
console.log(data);
```

可见，原异步调用的回调函数的`data`被函数直接返回，函数名需要改为`readFileSync`，其它参数不变。

如果同步读取文件发生错误，则需要用`try...catch`捕获该错误：

```js
try {
    var data = fs.readFileSync('sample.txt', 'utf-8');
    console.log(data);
} catch (err) {
    // 出错了
     console.log(err);
}
```

### 1.1.2 写文件

将数据写入文件是通过`fs.writeFile()`实现的：

```js
var fs = require('fs');

fs.writeFile('./write.txt', data, function (err) {
    if (err) console.log(err)
    console.log('写入完毕');
});
```

`writeFile()`的参数依次为文件名、数据和回调函数。如果传入的数据是String，默认按UTF-8编码写入文本文件，如果传入的参数是`Buffer`，则写入的是二进制文件。回调函数由于只关心成功与否，因此只需要一个`err`参数。

说到这里我想起了一个朋友问我的一个问题，细心的同学应该会发现这个writeFile会将原文本覆盖，所以，有没有什么好的方法能追加或者不覆盖呢？答案是肯定的，肯定会有的。

```js
//追加文本内容 方法1
const data = "hello,nihao"
fs.appendFile('./write.txt', data, function (err) {
    if (err) console.log(err)
    console.log('写入完毕');
});
```

```js
//追加文本内容 方法2
const data = "hello,nihao";
fs.open('ttt.txt', 'a', function (err, fd) {
    if (err) {
        console.log(err);
    } else {
        console.log('open');
        fs.writeFile(fd, data, function (err) {
            if (err) {
                console.log(err);
            } else {
                console.log('ok new');
            }
        });
    }
});
```



和`readFile()`类似，`writeFile()`也有一个同步方法，叫`writeFileSync()`：

```js
var fs = require('fs');

var data = 'Hello, Node.js';
fs.writeFileSync('output.txt', data);
```

### 1.1.3 stat

如果我们要获取文件大小，创建时间等信息，可以使用`fs.stat()`，它返回一个`Stat`对象，能告诉我们文件或目录的详细信息：

```js
'use strict';

var fs = require('fs');

fs.stat('sample.txt', function (err, stat) {
    if (err) {
        console.log(err);
    } else {
        // 是否是文件:
        console.log('isFile: ' + stat.isFile());
        // 是否是目录:
        console.log('isDirectory: ' + stat.isDirectory());
        if (stat.isFile()) {
            // 文件大小:
            console.log('size: ' + stat.size);
            // 创建时间, Date对象:
            console.log('birth time: ' + stat.birthtime);
            // 修改时间, Date对象:
            console.log('modified time: ' + stat.mtime);
        }
    }
});
```

运行结果如下：

```js
isFile: true
isDirectory: false
size: 18100
birth time: Fri Dec 11 2019 09:43:41 GMT+0800 (CST)
modified time: Fri Dec 11 2019 12:09:00 GMT+0800 (CST)
```

`stat()`也有一个对应的同步函数`statSync()`

### 1.1.4 异步还是同步

在`fs`模块中，提供同步方法是为了方便使用。那我们到底是应该用异步方法还是同步方法呢？

由于Node环境执行的JavaScript代码是服务器端代码，所以，绝大部分需要在服务器运行期反复执行业务逻辑的代码，*必须使用异步代码*，否则，同步代码在执行时期，服务器将停止响应，因为JavaScript只有一个执行线程。

服务器启动时如果需要读取配置文件，或者结束时需要写入到状态文件时，可以使用同步代码，因为这些代码只在启动和结束时执行一次，不影响服务器正常运行时的异步执行。

##  1.2 path 模块

* 使用方法

  * 加载模块

    ```js
    // 使用核心模块之前，首先加载核心模块
    let path = require('path');
    // 或者
    const path = require('path');
    ```
    
   * 举例

     ```js
     const path = require('path');
     
     // extname -- 获取文件后缀
     console.log(path.extname('index.html')); // .html
     console.log(path.extname('index.coffee.md')); // .md
     
     // join -- 智能拼接路径
     console.log(path.join('/a', 'b', 'c')); // \a\b\c
     console.log(path.join('a', 'b', 'c')); // a\b\c
     console.log(path.join('/a', '/b/../c')); // \a\c
     console.log(path.join('/a', 'b', 'index.html')); // \a\b\index.html
     console.log(path.join(__dirname, 'a', 'index.html')); // 得到一个绝对路径
     ```

## 1.3 Stream模块

`stream`是Node.js提供的又一个仅在服务区端可用的模块，目的是支持“流”这种数据结构。

示例：

```js
var fs = require('fs');

// 打开一个流:
var rs = fs.createReadStream('sample.txt', 'utf-8');

rs.on('data', function (chunk) {
    console.log('DATA:')
    console.log(chunk);
});

rs.on('end', function () {
    console.log('END');
});

rs.on('error', function (err) {
    console.log('ERROR: ' + err);
});
```

要注意，`data`事件可能会有多次，每次传递的`chunk`是流的一部分数据。

要以流的形式写入文件，只需要不断调用`write()`方法，最后以`end()`结束：

```js
var fs = require('fs');

var ws1 = fs.createWriteStream('output1.txt', 'utf-8');
ws1.write('使用Stream写入文本数据...\n');
ws1.write('END.');
ws1.end();

var ws2 = fs.createWriteStream('output2.txt');
ws2.write(new Buffer('使用Stream写入二进制数据...\n', 'utf-8'));
ws2.write(new Buffer('END.', 'utf-8'));
ws2.end();
```

所有可以读取数据的流都继承自`stream.Readable`，所有可以写入的流都继承自`stream.Writable`。

### pipe

就像可以把两个水管串成一个更长的水管一样，两个流也可以串起来。一个`Readable`流和一个`Writable`流串起来后，所有的数据自动从`Readable`流进入`Writable`流，这种操作叫`pipe`。

在Node.js中，`Readable`流有一个`pipe()`方法，就是用来干这件事的。

让我们用`pipe()`把一个文件流和另一个文件流串起来，这样源文件的所有数据就自动写入到目标文件里了，所以，这实际上是一个复制文件的程序：

```js
var fs = require('fs');

var rs = fs.createReadStream('sample.txt');
var ws = fs.createWriteStream('copied.txt');

rs.pipe(ws);
```

默认情况下，当`Readable`流的数据读取完毕，`end`事件触发后，将自动关闭`Writable`流。如果我们不希望自动关闭`Writable`流，需要传入参数：

```js
readable.pipe(writable, { end: false });
```

##  1.4 url模块

遗留的API

* 加载模块

   ```js
  const url = require('url');
  ```

* 使用方法：

   ```js
   let myURL = url.parse('/test.html?id=11&age=22'); // 返回一个包含url各个部分的对象
   ```

* 新方法（必须传递一个**完整的**url）

   ```js
   // 直接提供一个完整的url
   let myURL = new URL('http://www.xxx.com/test.html?id=11&age=22');
   // 或
   // 提供两个参数，一是文件路径及参数部分，二是域名，总之，二者组合必须是完整的url
   let myURL = new URL('/test.html?id=11&age=22', 'http://www.xxx.com');
   
   // 得到的myURL是一个对象，包含url中的各个部分
   // 如果需要解析参数部分，则使用querystring模块，或使用URL的一个子对象searchParams中的get方法
   let age = myURL.searchParams.get('age')； // 22
   ```

## 1.5 querystring模块

* 处理查询字符串（请求参数）的模块

* 使用方法：

  * 加载模块：

    ```js
    const querystring = require('querystring');
    ```

  * 调用querystring模块中的方法
  
    ```JS
    // parse -- 将查询字符串解析成JS对象
    console.log(querystring.parse('id=1&name=zs&age=20')); 
    // { id: '1', name: 'zs', age: '20' }
    
    // stringify -- 将JS对象转成查询字符串
    console.log(querystring.stringify({ id: '1', name: 'zs', age: '20' }));
    // id=1&name=zs&age=20
    ```
  
# 二.搭建一个服务器

## 2.1 流程：

```JS
// 1. 加载http模块
const http = require('http');

// 2. 创建服务对象，一般命名为server
const server = http.createServer(); // create创建、server服务器

// 3. 给server对象注册请求（request）事件，监听浏览器的请求。只要有浏览器的请求，就会触发该事件
server.on('request', () => {
    // 只要有浏览器的请求，就会触发该事件
    console.log('我发现你的请求了，但是不想搭理你');
});

// 4. 设置端口，开启服务
server.listen(3000, () => {
    console.log('服务器启动了');
});
```



* res形参：
  * res：response，响应；它是一个对象；它包含了所有和响应相关的信息
  * 响应对象，服务器给浏览器返回的响应内容，可以通过该对象设置
  * res.write()  设置响应体（返回给浏览器的内容）的内容，可以多次调用，但是只调用write不会做出响应，发送响应要调用 end()
  *  res.end()    把响应报文（响应行、响应头、响应体）发送给浏览器
  * res.setHeader()  设置响应头，比如设置响应体的编码
  * res.statusCode 设置状态码
  * res.writeHead(200, {响应头})
* req形参：
  *  req：request，请求；它是一个对象；它包含了所有和请求相关的信息 
  * req.url ----  获取请求的url
  * req.method --- 获取请求的方式，值为GET或POST
  * req.headers -- 所有的请求头
## 2.2 处理post请求方式： 

思路：创建连接方式与之前的思路是一样的，区别在于服务器接收数据的时候：

```js
// 有请求过来，就会触发这个事件。请求包含GET和POST两种请求
server.on('request', (req, res) => {
    // 当有数据提交给服务器之后。
    // 服务器接收提交过来的数据；并且保存（json文件中，或者数据库中）
    /**
     * 1. 创建一个空字符串，用于保存提交过来的数据
     * 2. 给req注册data事件，只要有数据提交过来，就会触发；用于接收提交过来的数据；数据比较大的话，是分块接收的
     * 3. 给req注册end事件，当完全接收了提交过来的数据，就会触发
     */
    let str = ''; // 定义一个用于保存数据的空字符串
    req.on('data', (chunk) => {
        // chunk -- 块
        str += chunk;
    });
    req.on('end', () => {
        console.log(str); // id=1&name=zs&age=43
    });
});
```

## 2.3 处理静态资源：

因为在html的页面中存在着多样的数据类型，我们在处理的时候需要对多种类型的数据进行设置`Content-Type`

举例：

````JS
response.setHeader('Content-Type', 'text/css');
````

可能存在的类型：

* .html：text/html
* .css：text/css
* .js：application/javascript
* .jpg：image/jpg

那么问题来了 ，这些类型我们不可能一个一个的设置，引入第三方模块：

```js
 const mime = require('mime'); // mime的作用，可以根据文件名生成合理的类型
 let type = mime.getType(req.url);
 res.setHeader('Content-Type', type);
```

## 2.4 第三方模块

在"楼上"处理静态资源处，提到了第三方模块，那究竟什么是第三方模块呢，又能做什么呢？

说白了：

* 使用第三方模块，多数都是为了简化开发，将复杂的代码封装后，形成的模块

安装方式：

* npm（node package manager）node包管理器

* 全局安装：

  * ```bash
    # 安装语法
    npm install -g 模块名
    # 新版Mac系统，如果提示权限不足，可以使用 sudo su npm install -g 模块名
    注意：全局安装不能使用`require()`
    ```

* 局部（本地）安装

  * ```bash
    与全局安装语法一致，将-g取消即可，
    # 安装语法
    npm install 模块名
    ```

### 2.4.1 比较有用的第三方模块

```bash
npm install mime        // mime的作用，可以根据文件名生成合理的类型
npm install -g nodemon  //用于自动启动服务的工具
npm install -g express  //基于Node.js快速开发的web框架
npm install body-parser //接收POST请求体的中间件
```

# 三.express框架

##  介绍



什么是express:Express 是一个基于 Node.js 平台，快速、开放、极简的 **web 开发框架**。

## 3.1 创建一个服务器的基本流程：

文字描述：

* 加载 express 模块
* 创建 express 服务器
* 开启服务器
* 监听浏览器请求并进行处理

代码描述：

```JS
// 使用express 搭建web服务器
// 1) 加载 express 模块
const express = require('express');

// 2) 创建 express 服务器
const app = express();

// 3) 开启服务器
app.listen(4000, () => console.log('express服务器开始工作了'));

// 4) 监听浏览器请求并进行处理

app.get('GET请求的地址', 处理函数);
或
app.post('POST请求的地址', 处理函数);
```

## 3.2 express新增的API

 1) express框架封装了一些额外的API（例如:send），可以让我们更方便的构造Web服务器

​	1.1 ）使用send方法响应数据的话，会自动设置content-type。但有时候会错误设置

​	1.2 ）注意send不能直接响应数字，需要加引号，否则会将数字当做响应状态码处理

 2) 浏览器请求的每一个url地址都会由一个独立方法接收并处理，没有了 if ... else if ... else 这样的分支，程序结构
     更加清晰

3）sendFile(文件路径); -- 功能是读取文件，并将读取到的结果响应给浏览器。它的参数必须是绝对路径。

## 3.3 中间件

介绍与使用:

* 中间件就是一个函数，中间件函数要当做 `app.use();` 的参数，这样来使用
* 中间件函数中有三个基本参数， req、res、next
* req就是请求相关的对象，它和后面用到的req对象是一个对象
* res就是响应相关的对象，它和后面用到的res对象也是一个对象
* next：它是一个函数，调用它将会跳出当前的中间件函数，执行后续代码；如果不调用next，则会在当前中间件卡住

## 3.4 中间件处理静态资源：

自定义中间件的代码：

```js
// 下面定义中间件，处理所有的文件（html、css、js、png等）
app.use((req, res, next) => {
    // console.log(req.url);
    // 判断是否有浏览器请求的文件，如果有，则读取并响应；如果没有则next
    const fs = require('fs');
    let filename = __dirname + '/public' + req.url;
    fs.access(filename, (err) => {
        if (err) {
            next();
        } else {
            res.sendFile(filename);
        }
    });   
});
```



express框架自带了更好的方法，我们来看一下:

```js
// 通过如下代码就可以将 public 目录下的图片、CSS 文件、JavaScript 文件对外开放访问了
app.use(express.static('public'));
```

## 3.5 body-parser中间件处理post提交数据

接收POST请求体的中间件(body-parser)

安装:

```bash
npm install body-parser
```

自定义中间件的代码：

```js
//自我封装的方法
app.use((req, res, next) => {
    // 判断，看是否是POST方式的请求
    if (req.method === 'POST') {
        // 这里的代码和之前一样，还是接收数据
        let str = '';
        req.on('data', (chunk) => {
            str += chunk;
        });
        req.on('end', () => {
            // 将接收到的数据，赋值给req.body
            // req.body属性本来不存在，是自定义的，你也可以用其他的名字
            req.body = querystring.parse(str);
            next();
        });
    } else {
        // 不是POST方式的请求，继续向下走
        next();
    }
})
```

body-parser 代码:

```js
// 如果请求头的 Content-Type为application/x-www-form-urlencoded，则将请求体赋值给req.body
app.use(bodyParser.urlencoded({extended: false})); // extended: false 表示将接收的数据用querystring模块处理成对象
```

# 四.mysql模块

## 4.1 简介

* mysql是一个第三方的模块，主要用来操作mysql数据库，对数据库进行增删改查等操作

安装：

```BASH
npm i mysql
```

curd:

```bash
curd: 就代表数据库的增删改查
c: create 就是添加 （增）
u: update 就是修改 （改）
r: read 就是查询 （查）
d: delete 就是删除 （删）
```

##  4.2 流程 （创建步骤）

> 1) 加载 MySQL 模块
>
> 2) 创建 MySQL 连接对象
>
> 3) 连接 MySQL 服务器
>
> 4) 执行SQL语句           
>
> 5) 关闭链接           

代码实现：

```JS
// 1. 加载mysql模块
const mysql = require('mysql');
// 2. 创建连接对象（设置连接参数）
const conn = mysql.createConnection({
    // 属性：值
    host: 'localhost',
    port: 3306,
    user: 'root',
    password: '',
    database: '*****' //填写你要连接的数据库
    multipleStatements: true // 一次性执行多条sql  （可选参数）
});
// 3. 连接到MySQL服务器
conn.connect();
// 4. 完成查询（增删改查）
//**********************************查询****************************
//占位符查询（第二个参数处可以传入数组，当为多个参数的时候）
let sql = 'select * from 数据库 where id<?';
conn.query(sql,3,(err,result)=>{
    if(err) throw err;
    console.log(result)
}); 
//**********************************添加****************************
//添加操作
let sql = 'insert into 数据库 set ?';
let values = {
    name:"泰达米尔",
    nickname:"蛮王",
    sex:"男",
    age:32,
};
//**********************************修改****************************
//修改
let sql = 'update 数据库 set ? where id = ?';
let values = {
    skill: '时光倒流',
    sex: '男'
}
conn.query(sql, [values, 36], (err, result) => {
    if (err) {
        console.log('修改失败');
    } else {
        console.log('修改成功');
    }
});
//**********************************删除****************************
// 4. 完成删除
let sql = 'delete from 数据库 where 字段 = ?';
conn.query(sql, 36, (err, result) => {
    if (err) {
        console.log('删除失败');
    } else {
        console.log('删除成功');
    }
});
//conn.query(sql语句,[sql语句中占位符的值],获取查询结果的回调函数);
conn.query(sql, values,(err, result) => {
    if (err) throw err;
    console.log(result)
});
// 5. 关闭连接，释放资源
conn.end();
```

# 五.模块化

## 5.1 Node中的模块化

* Node属于CommonJS标准

模块化的好处：

* 使用模块化可以很好的解决变量、函数名冲突问题，也能灵活的解决文件依赖问题
* 没有模块化，不允许一个js文件引入另外的JS，有了模块化，就允许一个js文件引入其他的js文件

## 5.2 作用域

局部作用域（模块的作用域）：

- 一个js文件就是一个模块
- 在一个js文件中定义的属性（变量、常量）和方法默认都只能在当前js文件中使用

全局作用域：

- 在js文件中声明的属性和方法如果都挂载到global对象下；当其他js文件导入该模块后，就能使用该模块下的属性和方法了。

  bb.js 中定义一些变量，并且把变量当做global的属性：

  ```js
  // 定义一些变量
  let abc = 'hello';
  let fn = (x, y) => {
      console.log(x + y);
  }
  // 把abc和fn当做global的属性
  global.abc = abc;
  global.fn = fn;
  ```

  aa.js 中，通过require加载另外的js文件，就可以使用bb中定义的变量了：

  ```js
  // 加载 b.js
  require('./bb.js');
  console.log(abc); // hello
  fn(3, 4); // 7
  ```

  **这个方案可以实现模块化，但是可能会造成全局环境污染**。

## 5.3 module.exports 导出属性和方法

- 将变量、对象、函数等挂载到global对象上并不推荐，因为容易造成变量污染。
- 推荐使用 module.exports 导出模块中定义好的变量、对象、方法
- 使用require加载（导入）模块后，就能使用模块中定义好的变量、对象、方法了

A.js 中定义一些变量，然后使用 `module.exports导出`：

```js
// 定义一些变量
let abc = 'hello';
let fn = (x, y) => {
    console.log(x + y);
};
// Node提供一套方案：
// 使用 module.exports 来导出模块（导出的只能是对象或函数）
module.exports = {
    abc: abc,
    fn: fn
};
--------------------------------
当函数的键名与变量名一致的时候可以简写成：
module.exports = {
    abc,fn
};
```

B.js 加载 A.js ，然后就得到了 A中导出的对象：

- **导入自己定义的js文件，必须加  `./`**

```js
// 导入模块(基本上和之前加载模块一样)
const bbb = require('./bbb.js');
// console.log(bbb); // { abc: 'hello', fn: [Function: fn] }
console.log(bbb.abc); // hello
bbb.fn(7, 8); // 15
```

## 5.4 封装db.js

实现原理：

* 1.加载mysql模块
*  2.创建连接对象，设置连接参数
* 3.连接mysql数据库
*  4.查询
* 5.关闭连接

将这5步骤封装到一个模块中，利用module.exports导出函数，随之在新的js页面加载此js调用即可。

###  5.4.1 代码实现(db.js)

```js
function db(sql, values, cb) {
    // 1.加载mysql模块
    const mysql = require('mysql');
    // 2.创建连接对象，设置连接参数
    const conn = mysql.createConnection({
        host: 'localhost',
        port: '3306',
        user: 'root',
        password: 'root',
        database: 'heroes',
        multipleStatements: true,
    });
    // 3.连接mysql数据库
    conn.connect();
    // 4.查询
    conn.query(sql, values, cb)
    // 5.关闭连接
    conn.end();
}

//模块到出
module.exports = db;
```

### 5.4.2 测试db.js

```js
//加载封装好的db.js
//切记这里一定要带 ./ 这个位置之前已经说明过，切记切记
const db = require('./01.db');

db('select id,name from heroes', null, (err, result) => {
    if (err) throw err;
    console.log(result);
});
```

#  六.multer模块

## 6.1 简介

* multer 是基于express框架的一个实现文件上传功能的中间件。
* 查看：<https://github.com/expressjs/multer>

##  6.2 配置

 配置方式1：

```js
// 加载模块
const multer  = require('multer');

// 配置
/*
1. 调用multer函数，需要给他传递一个对象，来对上传的文件进行配置
2. dest：表示目标，在这里表示文件上传的保存位置。下面设置上传的文件保存在当前文件夹的myup子文件夹中
3. 设置dest为一个目录后，如果该目录不存在，还可以自行创建该文件夹。
4. 得到的myupload是一个对象，该对象的几个方法都可以当做中间件，可以在添加或注册的时候使用
*/
const myupload = multer({
  dest: __dirname + '/myup'
});
```

配置方式2：

```js
// 加载模块
const multer  = require('multer');

// 配置
/*
1. 仍然是调用multer函数，给它传递一个对象进行配置
2. storage：文件的存储引擎，multer内置了磁盘存储(DiskStorage)和内存存储(MemoryStorage)
3. 下面使用的是diskStorage磁盘存储引擎，及它的配置
4. destination表示上传文件的存储路径，可以是一个函数，也可以是一个字符串
   如果用函数设置此项，函数的三个形参，分别表示请求的req对象、表示上传文件的file对象和一个回调函数
   无论使用函数还是字符串设置destination，都不会自动创建文件夹
5. filename 用于确定文件夹中的文件名的确定。 如果没有设置 filename，每个文件将设置为一个随机文件    名，并且是没有扩展名的。
6. 下面配置中，回调函数中的file形参表示上传的文件对象，对象的属性见后面的表
*/
const myupload = multer({
  storage: multer.diskStorage({
    destination: function (req, file, cb) {
      // cb是回调函数，用它设置上传路径，null的位置可以写成err，用于获取错误，使用null表示不获取错误
      cb(null, __dirname + '/abcd')
    },
    // destination: __dirname + '/cdef',
    filename: function (req, file, cb) {
      cb(null, file.fieldname + '-' + Date.now())
    }
  })
});
```

上面配置 destination和filename函数的file参数表示上传的文件对象，该对象具有以下属性：

| Key            | Description                     |
| -------------- | ------------------------------- |
| `fieldname`    | Field name 由表单指定           |
| `originalname` | 用户计算机上的文件的名称        |
| `encoding`     | 文件编码                        |
| `mimetype`     | 文件的 MIME 类型                |
| `size`         | 文件大小（字节单位）            |
| `destination`  | 保存路径                        |
| `filename`     | 保存在 `destination` 中的文件名 |
| `path`         | 已上传文件的完整路径            |

## 6.3 使用

在有文件上传的请求处理中，使用中间件即可

```JS
/////////////////////////////// 1. 单文件上传 ////////////////////////
// 单文件上传使用方式（形如：<input type="file" name="pic" />）
/*
在下面的 /abc 接口中，使用myupload.single() 方法作为该接口的中间件
myupload.single() 的参数 pic 为 表单项的name值
因为是单文件上传，所以在函数内部可以通过req.file 来获取上传的文件对象
*/
app.post('/abc', myupload.single('pic'), (req, res) => {
  console.log(req.file); // 上传的文件对象
  console.log(req.body); // 表单的文本信息
  res.send('文件上传成功');
});

/////////////////////////////// 2. 多文件上传 ////////////////////////
// 一个可以多文件上传的文件域（形如：<input type="file" multiple name="pic" />）
/*
在下面的 /abc 接口中，使用myupload.array() 方法作为该接口的中间件
myupload.array() 的参数 pic 为 表单项的name值， 2 表示允许上传的文件个数不能超过2个
因为是多文件上传，所以在函数内部可以通过req.files 来获取上传的文件对象
*/
app.post('/abc', myupload.array('pic', 2), (req, res) => {
  console.log(req.files);
  console.log(req.body);
  res.send('文件上传成功');
});


/////////////////////////////// 3. 多个文件域 ////////////////////////
// 有多个文件域 (形如：<input type="file" multiple name="pic1" /><input type="file" multiple name="pic2" />)
/*
在下面的 /abc 接口中，使用myupload.fields() 方法作为该接口的中间件
myupload.fields() 的参数 为一个数组，数组中的每个单元对应着一个文件域，name为文件域name属性值，maxCount表示该文件域可以选择的文件个数最大值
因为是多文件上传，所以在函数内部可以通过req.files 来获取上传的文件对象
*/
let up = upload.fields([
    { name: 'pic1', maxCount: 1 }, 
    { name: 'pic2', maxCount: 8 }
]);
app.post('/abc', up, (req, res) => {
  console.log(req.files);
  console.log(req.body);
  res.send('文件上传成功');
});
```

#  七 .会话技术

## 7.1 cookie

### 7.1.1 简介：

* 将数据持久化保存到**客户端**（浏览器）
* cookie是将数据持久化存储到客户端的一种技术。
* 网站可以将数据写到浏览器中， 一个网站最多能在一个浏览器写20个cookie。现在的浏览器能写的更多
* 一个浏览器能够设置的总cookie数最多为300个，每个不能超过4kb。
* cookie既能保存在文件中，也能保存在内存中。
* 可以通过浏览器查看某个网站的cookie

### 7.1.2 设置cookie

- 核心： cookie是服务端设置的，随着响应头返回给浏览器的信息，浏览器要将这些信息记录下来

- 如何设置cookie

  - 使用 res.setHeader
  - 使用res.writeHead
  - 如果使用的是express框架，可以使用res.set()方法

- cookie设置格式：`key=value;expires=time`

  - key: cookie的名称
  - value： 名称对应的值
  - expires： 有效期

  ```js
  // 所有的设置cookie，都必须写到一个请求中
  
  //1. 使用 http模块中的setHeader 方法，它可以和sendFile一起使用
  res.setHeader('set-cookie', 'id=101');                   //设置单个cookie
  res.setHeader('set-cookie', ['id=101', 'name=zs']);      //设置多个cookie
  
  //2. 使用 http模块中的writeHead 方法，它不能和sendFile及send一起使用
  res.writeHead(200, {
      'content-type': 'text/html;charset=utf-8',
      'set-cookie': ['type=10', 'name=my']
  })；
  
  //3. 使用 express模块中的set 方法，该方法是express封装的方法，它可以和sendFile一起使用
  res.set({
      'set-cookie':['goodsName=xiaomi 6', 'goodsPrice=3999']
  })；
  
  //4. 设置cookie时，指定有效期
  //注意：要使用UTC时间，使用 toUTCString()方法转换
  //设置有效期为 1小时
  const expiresTime = new Date(Date.now() + 3600000).toUTCString();
  res.set({
      'set-cookie':['goodsName=xiaomi 6;expires=' + expiresTime, 'goodsPrice=3999']
  })
  ```

  ### 7.1.3  读取cookie

  一旦网站在浏览器设置好cookie之后，浏览器**再访**问网站（任何页面）时，浏览器会将cookie信息随着请求头一起发送给服务器。所以我们在服务器端通过 `req.headers.cookie` 可以获取到cookie的信息。

  ```js
  const express = require('express');
  const app = express();
  app.listen(3000, () => console.log('启动了'));
  
  app.get('/captcha', (req, res) => {
      let vcode = '3Hj9';
      // 将验证码设置为cookie
      // cookie设置为响应头即可
      // res.setHeader('Set-Cookie', '名字=值');
      // res.setHeader('Set-Cookie', ['名字=值', '名字=值']);
      // res.setHeader('Set-Cookie', 'yzm=3Hj9');
  
      let time = new Date(Date.now() + 3600000).toUTCString();
      res.set({
          // 设置响应头
          'Content-Type': 'text/html; charset=utf-8',
          // 'Set-Cookie': ['age=20;expires=世界时间', 'name=zs', 'sex=F']
          'Set-Cookie': ['age=20;expires=' + time, 'name=zs', 'sex=F']
      });
  
      res.send('设置成功');
  });
  
  app.get('/login', (req, res) => {
      // 获取cookie，因为cookie中有yzm
      console.log(req.headers.cookie); // yzm=3Hj9
      res.send('获取成功');
  });
  ```

  ### 7.1.4 cookie有效期

  - 默认情况是，当关闭浏览器，cookie即消失
  - 如果设置了cookie的有效期（expires），则cookie会在指定的有效期内一直存在，无论浏览器是否关闭

  ```js
  // 设置cookie的有效期为1小时
  // 方式: 先找到一小时之后的时间戳，再转为utc时间
  // Date.now()： 获取当前时间点的时间戳 （毫秒数）
  // Date.now() + 3600000： 一小时之后的毫秒数
  // new Date(Date.now() + 3600000).toUTCString(): 将时间戳转为时间格式（UTC时间格式）
  let time = new Date(Date.now() + 3600000).toUTCString();
  // 设置cookie，有效期为1个小时
  res.set({
      'set-cookie': ['name=zs;expires='+time]
  });
  ```

  

## 7.2 session

### 7.2.1 简介：

*  将数据持久化保存到**服务器端**
* 因为cookie是保存在客户端的数据，不够安全，所以出现了session。
* session会将数据保存到服务器端（保存在文件、内存服务器或数据表中），安全性就可以得到保证。

### 7.2.2  设置/读取session

express设置session时，需要使用第三方模块 --- express-session

```bash
npm i express-session

# 这个模块也可以设置session     cookie-session
```

使用步骤：

1) 加载 express-session 模块

2) 将session注册为中间件（这样，当有请求过来的时候，都会先经过中间件）

3) 使用 `req.session` 对象设置/读取session

```JS
const express = require('express');
const app = express();
app.listen(3000, () => console.log('启动了'));

const session = require('express-session');
// 使用session中间件
app.use(session({ 
    secret: 'asre23s2323',  // 加密串，随便写
    cookie: { maxAge: 60000 }, // 过期时间，目前设置为 1分钟
    resave: false, // 重新保存
    saveUninitialized: false, // 即使未初始化也要保存
    // store: 保存位置 （文件、数据库中、memcache中......，保存到其他位置，还需要其他中间件）
}));

app.get('/captcha', (req, res) => {
    // 设置session
    // req.session.名字 = 值;
    req.session.yzm = 'abcd';
    req.session.isLogin = true;
    res.send('设置成功');
});

app.get('/login', (req, res) => {
    // 获取session
    console.log(req.session.yzm); // abcd
    console.log(req.session.isLogin); // true
    res.send('获取成功');
});
```

### 7.2.3 session有效期

- 当服务器关闭后，session消失
- express-session会将session保存在内存中，每次重启服务器时即使没有关闭浏览器session也会消失

> 学习阶段都是开发环境，服务器一会关闭了，一会开启了。
>
> 开发环境中，session是保存在内存中的，所以关闭服务器，session就消失了
>
> 生产环境中session的有效期要设置在配置项中，`cookie: {maxAge: 3600000}`，session应该保存到内存服务器中（memcache、Redis、MongoDB等）

### 7.2.4 删除session

核心： req.session.destroy()    销毁所有session

```js
// 删除所有session
req.session.destroy((err) => {
    if (err) {
        // 删除失败
    } else {
        // 删除成功
    }
});
```

### 7.2.5 session 的有效范围

在一个网站中设置了session，则整个网站都能找到这个session

## 7.3 cookie、session原理

cookie原理：

![1567952227485](C:\Users\56299\AppData\Roaming\Typora\typora-user-images\1567952227485.png)

session原理：

服务器端会为每个用户（浏览器）各自保存一个session（文件）

下次用户再来访问的时候，就不能确定该用户的session是哪一个了

所以当服务器保存session之后，会以cookie的形式告诉浏览器，你的session是哪一个

下次再来访问服务器的时候，浏览器就会带着它自己的session号去访问，服务器就可以找到对应的session了

![1567952258578](C:\Users\56299\AppData\Roaming\Typora\typora-user-images\1567952258578.png)

## 7.4 cookie和session的优缺点

- cookie：优点是节省服务器空间，缺点不安全。
  - 不需要特别安全的，使用cookie
  - 比如，视频观看的位置
- session：优点是安全，缺点需要服务器空间。
  - 安全性要求较高，使用session

## 八. 路由模块

### 8.1 什么是路由

路由（router），就是请求的接口地址

## 8.2 为什么要拆分

完成英雄管理案例之后，我们会发现app.js文件变的很大。

越大的文件越难于维护，所以我们需要将app.js文件进行拆分，让每个文件变的更小，功能更单一，这样就有利于项目的后期维护了。

原则： 让每个文件都更小，功能更单一

## 8.3 拆分路由一般规则

**将功能相同或相近的功能单独拆分成一个路由文件**

##  8.4  使用路由语法

在合适的位置创建 路由 文件，它也是一个js文件

举例：

* 比如将登陆和注册功能拆分成一个路由，则可以创建一个 `login.js` ，用于处理 `/reg` 、`/login` 等接口；
* 比如再创建一个 `student.js`，用于处理 `/getStudent`、 `/addStudent`、`/updateStudent` 等接口。

在login.js 中使用创建路由对象，将路由挂载到路由对象上

```JS
// 1. 加载express
const express = require('express');
// 2. 创建路由对象
const router = express.Router();

// 3. 将接口挂载到路由对象上
router.post('/login', (req, res) => {
    ....
});

router.get('/reg', (req, res) => {
    ....
});

...


// 4. 导出路由模块
module.exports = router;
```

然后在app.js 中加载路由，然后将其注册成中间件即可：

```JS
// 5. 加载路由模块，并注册成中间件
const router = require(__dirname + '/login.js');
app.use(router);
```

> 拆分路由后，需要从新整理代码，解决错误。策略是路由文件中使用了什么第三方模块，就需要加载第三方模块，否则报错。

> __dirname指向发生变化了，需要在app.js中，使用全局变量 global.rootPath 将 \_\_dirname 的值保存，其他路由文件使用rootPath代替\_\_dirname