## package.json 文件

### `npm init` 创建 package.json 文件

### `cat package.json` 展示 package.json 文件

### `rm package.json` 删除 package.json 文件

### `npm init -y` 全部选择默认选项创建 package.json 文件

## 两种模块化

### CommonJs or CJS

```js
const http = require('http')

module.exports = {
  name: 'Jose Cannon',
}
```

### ECMAScript Modules - ESM（2015 年）

```js
import http from 'http'

export const name = 'Josephine Stone'
```

### 将包指定为 ESM 的模块化

在 package.json 文件中

```json
{
  "type": "module"
}
```

## process.argv 获取用户传入的命令行参数

- 属性值是一个数组

```js
argv: ['D:\\nodejs\\node.exe', 'D:\\frontend_project\\nodejscmd\\index.js']
```

- 第一项是用于启动该进程的文件路径 `process.execPath`
- 第二项是该进程正在执行的 js 文件的完整路径
- 这个数组还可以的有其它更多项，表示启动该进程时传入的命令行参数

```
> node index.js hello world
[
  'D:\\nodejs\\node.exe',
  'D:\\frontend_project\\nodejscmd\\index.js',
  'hello',
  'wrold'
]
```

- 可以以字符串的方式传入值，这样可以传入带空格的命令行参数

```
node index.js "hello world"
[
  'D:\\nodejs\\node.exe',
  'D:\\frontend_project\\nodejscmd\\index.js',
  'hello world'
]
```

## readline 模块实现命令行和用户的交互能力

1. 引入 readline 模块

```js
import readline from 'ndoe:readline'
```

2. 监听标准输入

```js
const rl = readline.createInterface({
  terminal: true, // 表示在终端上活动
  input: process.stdin, // 表示将输入调整到标准输入
  output: process.stdout, // 表示将输出调整到标准输出
})

console.log('What is your name?')

let input = '' // 用于收集数据

// 监听键盘按下事件
rl.input.on('keypress', (event, rl) => {
  if (rl.name === 'return') {
    // 'return' 表示回车
    console.log(input)
  } else {
    input += event
  }
})
```

## readline/promises 模块实现命令行和用户交互行为

1. 引入模块

```js
import readline from 'node:readline/promises'

const rl = readline.createInterface({
  terminal: true,
  input: process.stdin,
  output: process.srdout,
})

const answer = await rl.question('What is your name?')

console.log(answer)

rl.close()
```

## nodejs 中使用 inquirer 第三方模块处理用户输入

1. 安装 `inquirer` 包

```
> npm i inquirer
```

2. 引入模块

```js
import inquirer from 'inquirer'
```

3. 不再使用 readline，而是使用 inquirer 来和用户交互

```js
async function askQuestion() {
  // 问题的答案会被收集在 answers 中
  const answers = await inquirer.prompt([
    {
      type: 'input', // 表示问题的类型（普通的问题，等待用户输入）
      name: 'name', // 表示问题的名字
      message: 'What is your name?', // 表示问题的内容
    },
    {
      type: 'list', // 列表问题，等待用户选择
      name: 'live',
      message: 'Where do your live?',
      choices: ['Sweden', 'Comoros', 'Bahrain'], // 可供用户选择的选项
    },
  ])

  console.log(answers.name)
  console.log(answers.live)
}
```

## 文件操作，从 json 文件中读取问题并解析

1. 定义一个 json 文件保存问题

```json
// 在 questions.json 文件中
[
  {
    "question": "Where do your name?",
    "answer": "xian",
    "id": 1
  }
]
```

2. 使用 `node:fs/promises` 模块读取文件

```js
async function askQuestion() {
  // 读取并解析 questions.json 文件中保存的问题数据
  const questionsData = JSON.parse((await fs.readFile('./questions.json')).toString())

  // ...
}
```

## 