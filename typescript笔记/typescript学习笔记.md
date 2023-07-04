## 1. ts开发环境搭建
1. 下载安装node.js
2. 全局安装ts：`npm i -g typescript`
3. 创建ts文件并使用`tsc <filename>`编译成js文件

## 2. 编译选项
* 自动编译文件
  * 编译文件时使用-w指令后，ts编译器会自动监视文件的变化，并在文件发生变化时重新编译
  * 指令`tsc <filename> -w`

* 自动编译整个项目
  * 如果直接使用`tsc`指令，可以将当前项目下的所有ts文件全部编译为js文件
  * 但直接使用`tsc`指令的前提是在项目根目录下创建一个tsconfig.json配置文件
  * 配置选项
    * include
      * 定义希望被编译的文件所在的目录
      * 默认值`["**/*"]`，即所有目录下的所有文件（两个`**`表示任意目录，一个`*`表示任意文件）
      * 示例：
        * ```json
            "include": ["src/**/*", "test/**/*"]
            ```
  * 上述示例中，所有的src和test下的目录和文件都会被编译
      
    * exclude
      * 定义需要排除在外的目录
      * 默认值`["node_modules", "bower_components", "json_packages"]`
      * 示例：
        * ```json
            "exclude": ["src/hello/**/*"]
        ```
      * 上述示例中，src/hello文件夹下的所有文件都不会被编译
      
    * extends
      * 定义被继承的json配置文件
      * 示例：
        * ```json
        "extends": "./config/base"
            ```
      * 上述示例中表示，将config目录下的base.json文件中的配置继承下来
      
    * files
      * 指定被编译的文件列表，只有文件很少时才会用到
      * 示例：
        * ```json
            "files": [
                "core.ts",
                "sys.ts",
            ......
            ]
            ```
      * 上述示例中，只有files数组中的文件才会被编译
      
    * compilerOptions
      * 编译器选项配置，是一个对象
      * 在compilerOptions中包含了多个子选项，用来完成对编译的配置
      * 项目选项
        * target
          * 设置ts代码编译的目标版本
          * 可选值：ES3（default），ES5，ES6/ES2015，......
          * 示例：
            * ```json
                  "compilerOptions": {
                      "target": "ES6"
                  }
              ```
            * 如上设置，代码将被编译为es6版本
        * module
          * 设置使用的模块化标准
          * 可选值：和target相似
          * 示例：
            * ```json
                "compilerOptions": {
                    "module": "commonjs"
                }
                ```
            * 上述示例表示，使用commonjs的模块化
        * lib
          
          * 指定代码运行所包含的库（运行环境），一般不用写
        * outDir
          * 编译完成的js文件的输出路径
          * 示例：
            * ```json
                "compilerOptions": {
                    "outDir": "dist"
                }
                ```
            * 表示将编译完成的js文件放在同级目录下的dist目录中
        * outFile
          * 将编译完的多个js文件合成为一个文件
          * 示例：
            * ```json
                "compilerOptions": {
                    "outFile": "dist/app.js"
                }
                ```
            * 上述代码中，表示将所有的编译完成的js代码合并到dist目录下的app.js中
        * allowJs和checkJs
          * 是否对js文件进行编译和检查，默认是false
          * 示例
            * ```json
                "compilerOptions": {
                    "allowJs": false,
                    "checkJs": false
                }
                ```
        * removeComments
          * 是否移除注释，默认值是false
          * 示例
            * ```json
                "compilerOptions": {
                    "removeComments": true
                }
                ```
        * noEmit
          
          * 是否生成编译文件
        * noEmitOnError
          
          * 在出现语法错误时是否生成编译文件

## 3. 类型声明
### 3.1 声明变量时指定类型
使用 `:` 指定
```typescript
let a: number;
// a = 'hello ts'; error: a is number
```
如果变量的声明和赋值是同时进行的，编译器会自动判断类型，可以省去指定类型
```typescript
let a = 10;
// a = 'hello ts'; error: a is number
```
给函数的参数和返回值指定类型（重要）
```typescript
function doSome(arg1: number): number {
    return arg1;
    // return arg1 + ''; error: should return a number
}

// doSome(''); error: arg1 is number
// const res: string = doSome(100); error: res is string
```
### 3.2 t中所拥有的类型
|  类型   |       例子       |             描述             |
| :-----: | :--------------: | :--------------------------: |
| number  |   1, -33, 2.5    |           任意数字           |
| string  | 'hello', "hello" |          任意字符串          |
| bollean |   true, false    |            布尔值            |
| 字面量  |      其本身      | 限制变量的值就是该字面量的值 |
|   any   |        *         |           任意类型           |
| unknown |        *         |        类型安全的any         |
|  void   | 空值或undefined  |     没有值或者undefined      |
|  never  |      没有值      | 不能是任何值，包括undefined  |
| object  |        {}        |         任意的js对象         |
|  array  |        []        |          任意js数组          |
|  tuple  |     [1, 2,]      |     元组，长度固定的数组     |
|  enum   |    enum{A, B}    |     枚举，ts中新增的元素     |

### 3.3 使用字面量声明类型，变量只能取字面量的值
```typescript
const str: 'hello' = 'hello';
// const str: 'hello' = 'ts'; error: can`t 'ts' for str, only 'hello'
```

### 3.4 联合类型声明
中间以 `|` 分隔，表示该变量可以取指定的多个类型的值
```typescript
let vara01: 'hello' | 'ts';
vara01 = 'hello';
vara01 = 'ts';
// vara = 'other'; error: vara01 only is 'hello' or 'ts'

let vara02: number | boolean;
vara02 = 10;
vara02 = true;
// vara02 = 'hello ts'; error: vara02 only is number or boolean
```
使用联合类型声明可以实现枚举

### 3.5 any
声明了any类型之后，编译器会放弃对这个类型的检查，**不建议**使用any
```typescript
// 显式any
let any01: any;
// 隐式any
let any02;
```

### 3.6 unknown
* 用法和any一样
* 区别：
  * any类型的值可以赋给别的变量且不会进行类型检查（不会报错）
    * ```typescript
        const vara01: any = 'hello';
        let vara02: number = vara01; // not error
        ```
  * unkonwn在**直接**赋给别的变量时会进行类型检查，即使unknown的值的类型和所赋变量的类型是符合的，也会报错 
    * ```typescript
        const vara01: unknown = 'hello';
        // let vara02: string = vara01; error: can`t unknown for string
        ```
  * 如果想要unknown能够赋值给其他变量，那么要手动进行类型检查，有两种方法：if判断，类型断言
    * ```typescript
        const vara01: unknown = 'hello'; 
        // 1. if judge
        if (typeof vara01 === 'string') {
          const vara02: string = vara01; // not error
        }
  
        // 2. type assert
        const vara02: string = vara01 as string;
        // or
        const vara03: string = <string>vara01;
        
        ```
### 3.7 void
主要用来设置函数的返回值，表示返回结果是空，如果声明了void类型，那么可以不写return语句
```typescript
// return void
function doSome(): void {

}
```
如果声明了void类型，并且写了return语句，并且返回值是undefined或null或没有，那么也是会返回void类型
```typescript
function dosome(): void {
    return;
    // return null;
    // return undefined;
}
```
如果没有声明void类型，并且没有return语句，那么默认返回是void类型
```typescript
// default return void
function doOther() {

}
```
如果没有声明void类型，并且有return语句，并且返回的不是null或undefined或者没有，那么编译器会自动判断返回值类型
```typescript
function doOther() {
    return 10; // auto judge return number
}
```

### 3.8 never
表示没有返回值（注意不是空，是什么都没有）
```typescript
function doSome(): never {
    thorw new Error('happen error');
}
```
拥有这个返回值的函数的主要作用是用来抛错的，抛错函数不需要有任何的返回值

### 3.9 object
一般用来限制对象内的属性个数和属性的类型
```typescript
const obj01: { name: string } = {
    name: 'zhangsan'
}
```
变量的值必须和声明的对象类型完全一致，否则会报错。如果在一个属性后面加上 `?` ，表示这个属性是可选的
```typescript
const obj02: { name: string, age?: number } = {
    name: 'zhangsan'
}
```
如果想要表示任意多个属性是可选的，使用`[propName: strnig]: any`
```typescript
let obj03: { name: string, [propName: string]: any };
obj03 = {
    name: 'zhangsan',
    age: 18,
    gender: 'man'
};
```
如果想声明函数类型，要写成箭头函数的形式
```typescript
// let doSome: (arg: <type>, ...) => <type>;
const sum: (a: number, b: number) => number = function (a, b) {
    return a + b;
}
```

### 3.10 array
声明方式有两种
```typescript
// 1. <type>[]
const arr01: number[] = [1, 2, 3];

// 2. Array<type>
const arr02: Array<number> = [1, 2, 3];
```

### 3.11 tuple
元组，固定长度的数组
```typescript
// let tuple01: [<type>, <type>, ...]; 
const tuple01: [string, number, boolean] = ['hello ts', 10, true];
```

### 3.12 enum
使用方式
```typescript
// first define an enum object
enum Gender = {
    MALE,
    FEMALE
}

// then declare enum type
const gender: Gender = Gender.MALE;
```

### 3.13 &类型声明
表示同时满足两个类型
```typescript
const person: { name: string } & { age: number } = {
    name: 'zhangsan',
    age: 18
}
```

### 3.14 类型别名
如果类型声明复杂且多次使用，可以给类型起一个别名
```typescript
type person = {
    name: string,
    age: number,
    gender?: Gender,
    getName: () => string,
    setName: (name: string) => void,
    [propName: string]: any
};

const person: person = {
    name: 'zhangsan',
    age: 18,
    gender: Gender.MALE,
    getName() {
        return this.name;
    },
    setName(name) {
        this.name = name;
    }
}
```

## 4. 面向对象
### 4.1 使用`class`关键字来定义类
```typescript
class Person {
    name: string = '' // default value
    age: number = 0
    static country: string = 'china' // static attribute
    readonly gender: Gender = Gender.MALE // read only attribute

    construcotr(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
```
### 抽象类
* 在普通类前面加上`abstract`关键字，就成了抽象类，抽象类只能用来被继承，不能创建对象
* 抽象类中可以定义抽象方法，抽象方法只能被定义在抽象类中，不能有具体实现，子类必须重写抽象方法
```typescript
// abstract class
abstract class Animal {
    abstract doSome(): void; // abstract function, only define in abstract class
}
```

### 接口
可以当成一个类型声明来使用
```typescript
type person =  {
    name: string,
    age: number
}

interface person {
    name: string
    age: number
}

const person: person {
    name: 'zhangsan',
    age: 18
}
```
接口可以重复定义，ts编译器会把所有重复定义的接口整合到一起
```typescript
interface person {
    name: string
}

interface person {
    age: number
}

const person: person {
    name: 'zhangsan',
    age: 18
}
```
接口最主要的用途是定义类的结构，规定类中应该有哪些属性，所有的属性不应该有值或实现
```typescript
interface AnimalInter {
    name: string

    doSome(): void;
}

class Animal implements AnimalInter {
    name: string

    constructor(name: string) {
        this.name = name;
    }

    doSome(): void {
        console.log('do some...');
    }
}
```

### 属性的封装
* 可以在类的属性前面加上`public`关键字，表示这个属性是一个公开的
* 如果加上一个`private`关键字，表示这个属性是私有的，外部访问不了
* 如果什么都不加，默认是public类型
```typescript
class Person {
    private name: string
    private age: number

    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    getName(): string {
        return this.name;
    }

    getAge(): number {
        return this.age;
    }

    setName(name: string): boolean {
        this.name = name;
        return true;
    }

    setAge(age: number): boolean {
        this.age = age;
        return true;
    }
}
```
* 一种更加方便的对私有属性读写的方式
```typescript
class Person {
    private _name: string
    private _age: number

    constructor(name: string, age: number) {
        this._name = name;
        this._age = age;
    }

    get name(): string {
        return this._name;
    }

    get age(): number {
        return this._age;
    }

    set name(name: string): boolean {
        this._name = name;
        return true;
    }

    set age(age: number): boolean {
        this._age = age;
        return true;
    }
}

// 使用时和直接使用普通属性的方式一样
new Person('zhangsan', 18).name = 'lisi';
```
* `protected`关键字，可以让属性在子类中被访问到

## 泛型
### 函数泛型
```typescript
function doSome<T, K>(a: T, b: K): T {
    return a;
}

// 使用时可以显式和隐式的使用
// 显式：
doSome<number, string>(10, 'hello ts');
// 隐式：ts编译器会自动推断出类型
doSome('hello ts', 10); // T是string，K是number
```
限定泛型的范围
```typescript
interface Length {
    length: number
}

// 表示T类型必须要继承Length这个接口
function dosome<T extends Length>(a: T): T {
    return a.length;
}

doSome<string>('hello ts'); // string这个类型实现了length这个接口
doSome({length: 10});
```
### 对象泛型
```typescript
class Person<T> {
    name: T

    constructor(name: T) {
        this.name = name;
    }
}

new Person<string>('zhangsan');
new Person('zhangsan'); // 会自动推断
```