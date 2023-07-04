# nodejs 笔记

---

## 1. 什么是 node.js

---

- node.js 是一个基于**Chrome v8**引擎的 javascript**运行环境**（相当于把 js 在浏览器中的运行环境提出来）
- 可以用来做后端开发
- 提供了内置的一些新的接口和内置对象，不包含 dom，bom 等 api
- 浏览器是 js 的前端运行环境，node.js 是 js 的后端运行环境

## 2. 使用 node 运行 js 代码

---

在终端里使用 `node <filename>` 即可执行 js 代码

## 3. fs 文件系统模块

---

### 3.1 什么是文件系统模块

文件系统模块是 node.js 内置的一个用来操作文件系统的模块，提供了一些列的方法和属性，可以满足用户对文件的读写要求

- fs.readFile() 用来读取文件的内容
- fs.writeFile() 用来写入文件的内容

使用之前要先导入 fs 模块

```javascript
const fs = require('fs');
```

### 3.2 读取文件内容

语法格式

```javascript
fs.readFile(path[, options], (err, data) => {
    ...
});

// path: 文件路径，字符串形式
// operation: （可选）以什么样的编码格式读取文件内容，默认是utf-8格式
// 第三个参数：回调函数，成功时可以通过这个函数拿到文件内容
//      err: 读取失败时的错误对象
//      data: 读取成功时的文件数据
```

### 3.3 写入文件内容

语法格式

```javascript
fs.writeFile(path, data[, options], err => {
    ...
});

// path: 文件路径字符串
// data: 要写入的数据
// options: （可选）以什么样的格式写入数据，默认是utf-8格式
// callback: 不论成功还是失败，都会调用这个回调函数
```

如果路径字符串对应的文件不存在，那么会自动创建一个，如果存在在，则会覆盖原来的文件内容

### 3.4 追加文件内容

语法格式

```javascript
fs.appendFile(path, data[, options], err => {
    ...
});

// 参数与writeFile()相同
```

## 4. path 路径模块

---

### 4.1 路径动态拼接错误的问题

- 在使用 fs 模块操作文件时，如果提供的路径是以`./`或者`../`开头的相对路径，很容易出现路径动态拼接错误的问题
- 原因：代码运行时会以运行 node 命令时所处的目录，动态拼接出文件按的完整目录
- 解决：提供一个完整的文件路径

### 4.2 如何提供完整的文件路径

- 使用`__dirname`和`__filename`
- \_\_dirname：表示本文件所在的文件夹的完整目录
- \_\_filename：表示本文件的完整目录

```javascript
// 表示当前文件夹下的xxx.json文件，可以这样写
const path = __dirname + '\\xxx.json';
```

### path 相关 api

使用之前要先导入 path 模块

```javascript
const path = require('path');
```

#### path.join()

将多个路径碎片拼接成一个完整路径

```javascript
path.join(...paths: string[]);

// 注意：当paths中有../时，会回到上层的目录（抵消之前的一层路径）
path.join('/a', '/b/c', '../', '/e'); // /a/b/e
```

#### path.basename()

从路径字符串中将最后一部分解析出来，一般是文件名

```javascript
path.basename(path: string);

// 可以使用第二个参数将扩展名移除掉
path.basename(path: string, extendName: string);
// 例：
path.basename('a/b/c/test.js', '.js'); // test
```

#### path.extname()

从路径字符串中获取文件扩展名

```javascript
path.extname(path: string);
```

## 5. http 模块

---

### 域名的概念（domain name）

- 域名是 ip 地址的直观名字，和 ip 地址一一对应，这个对应关系存放在域名解析服务器中(DNS: domain name server)
- 在使用域名访问网站时，会先到域名解析服务器中查询相关 ip，然后对对应的 ip 进行访问
- 例如：127.0.0.1 的域名时 localhost

### 创建最基本的 web 服务器

#### 基本步骤

1. 导入 http 模块

```javascript
const http = require('http');
```

2. 创建 web 服务器实例

```javascript
const server = http.createServer();
```

3. 为服务器绑定 request 事件，监听客户端请求

```javascript
server.on('request', (req, res) => {
  console.log('someone visit the server');
});
```

4. 启动服务器

```javascript
server.listen(3000, err => {
  if (!err) console.log('serve start success');
});
```

#### req 请求对象

如果想在请求的时间处理函数中访问与客户端相关的数据或属性，可以使用 req 参数

```javascript
server.on('request', (req, res) => {
  // req是与请求信息有关的对象
  // req.url是客户端请求的url
  // req.method是客户端请求的方式
  console.log(req.url);
  console.log(req.method);
});
```

#### res 响应对象

```javascript
server.on('request', (req, res) => {
  // res是与服务器相关的数据和信息的对象
  // res.end() 可以向客户端返回响应数据，并结束这次请求
  res.end('hello http');
});
```

## 6. 模块化

---

### 6.1 模块化的基本概念

- 模块化是指将一个复杂问题分解为若干模块，每一个模块都是可分解，可更换，可组合的单元
- 模块化的好处
  - 提高代码的复用性
  - 提高代码的可维护性
  - 可以实现按需加载

### 6.2 模块化规范

对代码进行模块化拆分与组合时应该遵循的规则，例如如何导入一个模块，如何导出一个模块

### 6.3 nodejs 中的模块化

nodejs 根据模块的来源不同，将模块分为 3 大类，分别是：

- 内置模块
- 自定义模块（一个用户创建的 js 文件就是一个自定义模块）
- 第三方模块

### 6.4 加载模块

使用 require() 方法可以动态的加载这三类模块

### 6.5 模块作用域

和函数作用域类似，在模块中定义的变量，方法等，只能在该模块中被访问，出了该模块之后就不能被访问到了

```javascript
// module 1
console.log('## the module was loaded');

const username = 'zhangsan';

function do_some() {
  console.log(username);
}

// ---------------------------------------

// module 2
const m = require('./module 1');

console.log(m);
// console.log(username); error: username is not define
// do_some();
```

### 6.6 如何暴露一个模块中的私有成员

1. module 对象

- 每个.js 自定义模块中都有一个 module 对象，它里面存储了和当前模块有关的信息
- 打印信息如下

```javascript
  Module {
    id: 'D:\\frontend_project\\nodejs\\_module\\define_module.js',
    path: 'D:\\frontend_project\\nodejs\\_module',
    exports: {},
    filename: 'D:\\frontend_project\\nodejs\\_module\\define_module.js',
    loaded: false,
    children: [],
    paths: [
      'D:\\frontend_project\\nodejs\\_module\\node_modules',
      'D:\\frontend_project\\nodejs\\node_modules',
      'D:\\frontend_project\\node_modules',
      'D:\\node_modules'
    ]
  }
```

2. module.exports 对象

- 一个模块在被导入时，本质上就是拿到 module.exports 这个属性所指向的对象
- 这个属性在默认情况下是一个空对象
- 在被导入的模块中可以设置 module.exports 的对象属性，或者直接将其替换为一个新的对象，可以将该模块中的私有成员向外暴露

```javascript
// module 1
const name = 'zhangsan';
function do_some() {
  /* ... */
}

// 设置module.exports的属性
module.exports.name = name;
module.exports.do_some = do_some;

// 或者将其替换为一个新的对象
module.exports = {
  name,
  do_some,
};

// ------------------------------

// module 2
const m = require('./module 1');

console.log(m); // { name: 'zhangsan', do_some: Function }
```

3. exports 对象

- 使用起来和 module.exports 是一样的，只是后者写法比较复杂，前者简化了而已
- 一个注意点：module.exports 和 exports 的原理是两者都指向了同一个对象，如果修改了其中任何一个的指向，那么最终导入结果是 module.exports 指向的那个对象

## 7. npm 与包

---

1. nodejs 中的**第三方模块**又叫做**包**
2. 包是由第三方个人或团队开发出来的，免费共别人使用
3. **包是由内置模块封装出来的**，提供了更高级，更方便的 api
4. 包搜索平台：https://www.npmjs.com
5. 包下载地址：https://registry.npmjs.org/（可以配置国内镜像：https://registry.npm.taobao.org）
6. 使用 `npm i <完整包名>` 来下载包到指定的项目
7. 使用 `npm i <完整包名>@<版本号>` 来安装指定版本的包，如果没有指定版本号，默认安装最新的包
8. 使用 `npm i <完整包名> -g` 来全局安装包

---

小知识：版本号是点分十进制的形式，例如：`2.24.0`

1. 第一个数字是大的更新，例如底层代码重构时会将其加一
2. 第二个数字是添加或删除某个功能时会将其加一
3. 修复 bug 时会将其加一

版本号提升规则：要前面的版本号增加了，后面的版本号要**归零**

---

9. 包管理配置文件 `package.json`

- npm 规定，在项目的根目录中，必须提供一个 package.json 的包管理配置文件，用来记录与项目有关的一些配置
  - 项目名称，版本号，描述等
  - 项目中都用到了哪些包
  - 哪些包只在开发期间用到
  - 哪些包在开发和部署期间都用到
- 在把项目上传到 github 时，由于第三方包是不用上传的，所以要在.gitignore 文件中配置，但是 package.json 文件要上传，告诉别人该项目都使用了哪些包
- 快速创建 package.json 文件：使用命令 `npm init -y`

10. 批量安装包

- 如果要一次性安装多个包，可以使用 `npm i <包名1> <包名2> ...`
- 如果要一次性安装 package.json 中的 dependencies 指定的所有包，可以使用 `npm install` ，这个命令会去查找 package.json 中的 dependencies 中的所有包并全部下载

11. 使用 `npm uninstall <package name>` 来卸载包
12. 使用 `npm i <package name> -D` 或者 `npm i <package name> --save-dev` 将包下载到 devDependencies 中，在这个节点中的包指在开发阶段会用到，项目上线后就不会用到了

13. 包的分类：

- 项目包：那些被下载到 node_modules 文件夹中的包是项目包
  - 开发依赖包：被记录到 devDependencies 中的包是开发依赖包
  - 核心依赖包：被记录到 dependencies 中的包是核心依赖包
- 全局包：在安装时提供一个 `-g` 命令，会将这个包安装到全局，全局包卸载时也要加 `-g` 命令
- 注意：只有工具性质的包，才有全局安装的必要

14. i5ting_toc 工具包

- 全局安装：`npm i -g i5ting_toc`
- 作用：将 md 文档转换为 html 文档
- 使用：在对应目录下执行命令：`i5ting_toc -f <filename> -o`

15. 规范的包结构

- 一个规范的包必须符合以下三点：
  - 包必须以**单独的目录**而存在
  - 包的顶级目录里必须包含 `package.json` 包管理配置文件
  - `package.json` 里必须包含 `name`, `version`, `main` 这三个配置，分别代表包的名字，版本，入口文件

## 8. 开发自己的包

---

### 8.1 初始化包的基本结构：添加三个文件

- package.json（包管理配置文件）

```javascript
{
  "name": "myself_node_package", // 包的名称
  "version": "1.0.0", // 包的版本号
  "description": "This is a first node package that was created by me", // 包的简短的描述信息
  "main": "index.js", // 包的入口文件
  "scripts": { // 包的测试脚本
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["myself", "dateFormat"], // 包的npm检索关键词
  "author": "zhangpan", // 包的作者
  "license": "ISC", // 包的开源许可协议
  "type": "module" // 包的模块化类型 ESM
}
```

- index.js（包的入口文件）
- README.md（包的说明文档）

### 8.2 终端登录 npm 官网

使用 `npn login` 依次输入自己的用户名，密码，邮箱后，即可登录成功，注意：在使用 `npm login` 之前，要把下载 npm 包的源修改为 npm 官方，否则发包将会不成功

### 8.3 发布包

进入包的根目录下，使用命令 `npm publish` 发布自己的包

### 8.4 删除已发布的包

使用命令 `npm unpublish <package name> --force` 可以已发布的删除包，注意：只能在包发布的 72 小时内删除，删除后 24 小时内不允许重新发布包

## 9. 模块加载机制

---

### 优先从缓存中加载

**模块在第一次加载后会被缓存**，这意味着即使多次导入包，也不会导致模块的代码被多次执行<br>
注意：不论是内置模块，还是第三方模块，他们都会优先从缓存加载，从而提高模块的加载效率

### 内置模块的加载机制

内置模块的加载优先级是最高的，如果和其他模块重名，内置模块优先

### 自定义模块的加载机制

如果加载时没有指定路径标识符，那么会被当成内置或者第三方模块加载

```javascript
// import * as m from './pkg_name.js';
import * as m from 'pkg_name.js';
```

### 第三方模块加载机制

如果传递的参数既不是一个内置模块，也不是一个路径（以 ./ 或 ../ 开头），那么 nodejs 会从当前模块的父目录开始，尝试从 node_modules 文件夹中加载第三方模块，如果没有找到 node_modules，或者在 node_moudels 文件夹中没有找到对应模块，那么会到上层目录进行同样的操作，直到找到或者到文件系统的根目录

### 目录作为一个模块

- 将目录作为一个模块加载时，nodejs 会先找这个目录中的 package.json 文件，查询其中的 main 配置，找到入口文件进行加载
- 如果没找到 package.json 或者没有 main 配置项，nodejs 会去找该目录下的 index.js 文件进行导入
- 如果还没有 index.js 文件，那么会报错：can not find the module

## 10. Express

---

### 10.1 什么是 Express

Express 是基于 Node.js 平台的一个快速，开放，极简的 web 开发框架

### 10.2 Express 本质

Express 是一个 Node.js 的第三方包

### 10.3 如何使用 Express

1. 先下载安装 Express

```
npm i express
```

2. 创建一个最基本的 web server

```javascript
import express from 'express'; // 引入 Express

const app = express(); // 创建 app

// 监听80端口
app.listen(80, () => {
  console.log('server is lintening 80...');
});
```

### 10.4 监听 GET 和 POST 请求

- 通过 `app.get()` 方法可以监听客户端的 GET 请求

```javascript
// 格式如下
app.get('URL', (req, res) => {
  // ...
});

/*
参数：
  URL: 请求的URL
  (req, res) => {//...}: 请求处理函数
    req: 请求对象（包含了与请求相关的属性和方法）
    res: 响应对象（包含了与响应相关的属性和方法）
*/
```

- 通过 `app.post()` 方法可以监听客户端的 POST 请求，用法和 GET 请求一样
- 在请求处理函数中，可以通过 `res.send('json 字符串')` 方法将处理好的数据传送给客户端

### 10.5 获取客户端请求中的 query 参数

- 可以通过 `req.query` 对象拿到客户端的 query 参数
- `req.query` 默认是一个空对象

```javascript
// /user?name=zhangsan&age=18
// req.query:
{
  name: 'zhangsan',
  age: 18,
}
```

### 10.6 获取客户端请求中的动态参数

- 可以通过 `req.params` 对象拿到客户端的动态参数
- `req.params` 默认是一个空对象

```javascript
// /user/123
app.get('/user/:id' (req, res) => {
  console.log(req.params); // {id: '123'}
});
```

### 10.7 托管静态资源

- 使用 `express.static()` 可以方便的创建一个静态资源服务器，代码如下

```javascript
app.use(express.static('静态资源目录'));
```

- 静态资源下的所有文件都可以直接被外界访问
- 如果要托管多个静态资源目录，可以多次调用 `app.use(expres.static('静态资源目录'))`
- 如果要挂载路径前缀，可以再传一个前缀参数

```javascript
app.use('/public', express.static('./public'));

/*
这样可以使用 http://localhost:5005/public/xxx.xx来访问静态资源
*/
```

### 10.8 小工具：nodemon

- 作用：可以自动重启服务器，不用修改代码之后手动重启
- 使用 `npm i nodemon -g` 将 nodemon 安装为全局
- 使用 `nodemon <filename>` 运行文件，可以让 nodemon 监视文件代码，一旦变动就重启服务器

### 10.9 Expresss 路由

1. 路由概念：**客户端请求**与**服务器处理函数**之间的一种**映射关系**
2. 路由的组成

- 请求的方式
- 请求的 URL
- 处理函数

```javascript
// 这就是一个路由
app.METHOD(PATH, HANDLER);
```

3. 路由的使用：

- 最简单的方式是直接将路由挂载到 app 实例上，这种方式很少用

```javascript
app.get('/', (req, res) => {...});
```

- 将路由抽离成单独的模块

  1. 创建路由模块对应的 js 文件
  2. 使用 `express.Router()` 创建路由对象
  3. 向路由对象上挂载路由
  4. 导出路由对象
  5. 使用 `app.use` 注册路由模块

- 给路由模块添加前缀

```javascript
// 在注册路由模块时添加前缀
app.use('/prefix', router);
```

### 10.10 Express 中间件

#### 中间件的概念：

业务逻辑的中间处理环节，叫做中间件（Middleware）

#### Express 中间件的调用流程

当一个请求到达 Express 服务器后，可以连续调用多个中间件，从而对这个请求进行**预处理**

#### Express 中间件格式

Express 中间件本质上是一个 function 处理函数，格式如下

```javascript
app.get('/', (req, res, next) => {
  next();
});

/*
如果一个路由的处理函数包含了第三个参数 next，那么这个路由的处理函数就是一个中间件
如果没有 next，参数，那它仅仅是一个路由的处理函数
*/
```

#### next() 参数作用

`next()` 函数是实现多个中间件连续调用的关键，它表示把流转关系转交给下一个中间件或路由

#### 定义中间件函数

1. 定义一个最简单的中间件函数

```javascript
const mw = function (req, res, next) {
  console.log('this is a middleware');
  next();
};
```

2. 全局生效的中间件

- 使用 `app.use(中间件函数)` 即可定义一个全局生效的中间件
- 全局中间件：所有的客户端请求都会先经过这个中间件
- 作用：
  - 可以作为一个装饰器：多个中间件和路由之间是共享同一份 `req` 和 `res` 的。这样的话可以在上游中间件对这两个对象进行修饰（添加或删除一些属性和方法等），供下游的中间件或路由来使用
  - 可以作为一个过滤器：对到达服务器的请求进行检验，不合法的请求将不会转发给下游的中间件或路由
- 定义多个全局中间件：可以多次使用 app.use()定义多个中间件，请求会按定义顺序依次经过这几个中间件

```javascript
const mw = function (req, res, next) {
  console.log('this is a middleware');
  next();
};

app.use(mw);

// 简化形式定义
app.use((req, res next) => {
  console.log('this is a middleware');
  next();
});
```

3. 局部生效的中间件

- 在路由的内部作为一个函数参数参数定义
- 局部中间件：只有匹配到该路由的请求才会进入这个中间件
- 作用和全局中间件的作用相似

```javascript
const mw = function (req, res, next) {
  console.log('this is a local middleware');
  next();
};

app.get('/', mw, (req, res) => {});

// 简写形式
app.get(
  '/',
  (req, res, next) => {
    console.log('this is a local middleware');
    next();
  },
  (req, res) => {
    // ...
  },
);
```

#### 中间件的几个注意事项

- 一定要在注册路由之前注册中间件
- 请求可以连续经过多个中间件
- 不要忘记调用 `next()` 函数
- 为了防止代码逻辑混乱，调用 `next()` 之后就不要再写其他代码了
- 连续调用多个中间件时，多个中间件和路由之间共享 `req` 和 `res` 对象

#### 中间件的分类

1. 应用级别的中间件

- 通过 `app.use()` 或者 `app.get()` 或者 `app.post()`，绑定到 app 实例上的中间件，叫做应用级别中间件（全局中间件和局部中间件）

2. 路由级别的中间件

- 绑定到 `express.Router()` 实例上的中间件叫做路由级别中间价，用法和应用级别中间件一样

```javascript
const router = express.Router();

// 全局中间件
router.use((req, res, next) => {
  next();
});

// 局部中间件
router.get(
  '/',
  (req, res, next) => {
    next();
  },
  (req, res) => {},
);
```

3. 错误级别的中间件

- 作用：专门用来捕获整个项目中发生的异常错误，从而防止整个项目异常崩溃的问题
- 格式:

```javascript
const mw_err = function (err, req, res, next) {
  // ...
};
```

- 用法的一个示例：

```javascript
app.get('/', (req, res) => {
  throw new Error('throw a error');
  res.send('hello express');
});

// 捕获异常的错误中间件
app.use((err, req, res, next) => {
  console.log(`happen an error: ${err.message}`);
  res.send('have an error in server');
});
```

- **错误级别中间件必须放在所有路由定义之后**

4. Express 内置的中间件

- `express.static` 快速托管静态资源
- `express.json` 解析 json 格式的请求体数据，有兼容性问题

```javascript
app.use(express.json());
```

- `express.urlencoded` 解析 URL-encoded 格式的请求体数据，有兼容性问题

```javascript
app.use(express.urlcoded({ extended: false }));
```

5. 第三方中间件

#### 自定义中间件

- `req` 的 `data` 事件可以监听是否有请求体数据，如果有则触发该事件，事件的回调函数参数可以接收到此次请求体的数据

- `req` 的 `end` 事件可以监听请求体数据是否接收完毕，如果接收完毕则触发该事件

- 使用 `on` 给 `req` 绑定事件

```javascript
app.use((req, res, next) => {
  let data_str = '';

  req.on('data', chunk => {
    data_str += chunk;
  });

  req.on('end', () => {
    // 接收完毕之后，data_str 就是整个请求体数据的字符串
    console.log(data_str);
  });
});
```

### 10.11 基于 cors 解决接口跨域问题

- 使用 cors 中间件解决跨域问题
1. 使用 `npm i cors` 安装 cros 中间件
2. 导入中间件 `import cors from 'cors';`
3. 使用 `app.use(cors());` 配置中间件

### 10.12 cors 相关的三个响应头

