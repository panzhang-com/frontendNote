# Electron 笔记

## 搭建 Electron 的开发环境

---

1. 安装 `node.js`
2. 在项目中安装开发版本的 Electron 或者全局安装

```
npm init -y
npm i electron --save-dev
```

3. 检测是否安装成功 `npx electron -v` 或者 `./node_moudles/.bin/electron -v`

## Hello World 第一个程序

---

1. 编写一个 `index.html` 作为主界面
2. 编写一个 `index.js` 或者 `main.js` 作为程序入口
3. 在入口 js 文件中引入 electron

```js
const electron = require('electron');
```

4. 创建 app 应用和 BrowserWindow 浏览器窗口

```js
const app = electron.app; // 主进程
const BrowserWindow = electron.BrowserWindow; // 用于创建窗口的构造函数
```

5. 再创建一个主窗口，当 app 打开时展示这个窗口，暂时等于 null

```js
const mainWindow = null;
```

6. 给 app 绑定 ready 事件

```js
app.on('ready', () => {
  // 当 app 打开时创建一个主窗口用于展示
  mainWindow = new BrowserWindow({ width: 300, heigth: 300 });

  // 将之前的 index.html 载入到这个窗口
  mainWindow.loadFile('./index.html');

  // 给 mainWindow 绑定一个关闭事件，当窗口关闭时，让 mainWindow 等于 null，避免内存泄露
  mainWindow.on('closed', () => {
    mainWindow = null;
  });
});
```

7. 在项目根目录下使用 `electron main.js` 或者 `electron .` 即可运行项目

## Electron 的运行流程

---

1. 进入 package.json 文件寻找入口文件 main.js
2. 进入 main.js 文件（主进程文件）
3. 读取页面布局和演示
4. IPC 执行任务和获取信息

## Electron 主进程预渲染进程

---

1. 主进程（main process）：main.js 就是一个主进程，一个 Electron 程序有且只有一个主进程
2. 渲染进程（render process）：每一个打开的窗口就是一个渲染进程，例如 index.html 就是一个渲染进程

## 在渲染进程中使用 nodejs

---

- 在创建渲染进程时，配置 `webPreferences`

```js
mainWindow = new BrowserWindow({
  width: 300,
  height: 300,
  webPreferences: {
    nodeIntegration: true,
    contextIsolation: false,
  },
});
mainWindow.loadFile('./index.html');
mainWindow.on('closed', () => {
  mainWindow = null;
});
```

## 在渲染进程中使用 BrowserWindow 创建子窗口

---

由于主进程和渲染进程不是使用相同模块，所以主进程中的 BrowserWindow 不能直接用在渲染进程中，但可以通过 remote 方式使用

1. 安装 remote

```
npm i -save @electron/remote
```

2. 在渲染进程创建之前进行一些配置

```js
let window = BrowserWinodw({
  width: 500,
  height: 500,
  webPreferences: {
    nodeIntegration: true,
    contextIsolation: false,
  },
});

// 能够在渲染进程中使用 BrowserWindow
require('@electron/remote/main').initialize();
require('@electron/remote/main').enable(window.webContents);

window.loadFile('./index.html');

// -------------- index.js ------------
const { BrowserWindow } = require('@electron/remote');
```


