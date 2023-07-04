### 35.12 编程式路由导航

借助$router，脱离了router-link，使跳转更加灵活，可以在任意地方指定要跳转的页面，写法：

```javascript
// 借助vue-router的api

// push模式跳转
this.$router.push({
	path: '/xxx',
	query: {
		a: valueA,
		b: valueB,
		...
	},
});
// 参数传递的是一个配置对象，这个对象和使用<router-link>的to属性的对象写法时传入的对象是一样的
---------------------------------------------------
// replace模式跳转
this.$router.replace({
	...
});
// 配置对象的写法和push是一样的
---------------------------------------------------
// 前进和后退
this.$router.forward();
this.$router.back();
// 不需要传参
---------------------------------------------------
// go方法，可以指定向前或者向后条几步
this.$router.go(number);
// number为正表示向前跳对应步数，为负表示想后跳对应步数
```

### 35.13 缓存路由组件

作用是让路由组件在被切出去时保持挂载，不被销毁。写法：

```javascript
// 在展示路由组件的地方使用<keep-alive>标签将其包裹起来
<keep-alive include="componentName">
	<router-view></router-view>
</keep-alive>
// include属性表示要将哪个组件保持挂载，如果不写的话表示将在这里展示的所有路由组件都保持挂载
---------------------------------
// 如果有多个路由组件想要保持挂载，include要写成数组形式
<keep-alive :include="[ 'componentNamw1', 'componentName2', ... ]">
	<router-view></router-view>
</keep-alive>
```

### 35.14 两个新的生命周期钩子

作用是捕获路由组件的激活状态，或者说是否切换到该路由组件

```javascript
// 1. activated()
// 当切换到路由组件，即路由组件被激活时调用一次该钩子
activated() {
	...
}
----------------------------------
// 2. deactivated()
// 当从该路由组件切走时，即路由组件失活时，调用一次该钩子
deactivated() {
	...
}
```

需要注意的是，这两个生命周期钩子是只有路由组件才有的，一般组件没有这个钩子

实际上还有最后一个生命周期钩子，是$nextTick()，在下一次页面重新渲染之后调用参数里的回调函数

```javascript
this.$nextTick(() => {
	...
});
```

### 35.15 路由守卫

#### 全局前置路由守卫：

作用：在**初始化时&切换路由之前**添加一个路由的跳转权限

```javascript
// 在index.js中：
const router = new Vue-Router({ ... })

router.beforeEach((to, from, next) => {
	if (to.fullName === '/xxx' || ...) {
		...
	}
});

export default router;
```

三个参数的作用：

to：要跳转的目标的信息，是一个对象

```javascript
{
	fullPath: "/xxx"
	hash: ""
	matched: [{…}]
	meta: {}
	name: undefined
	params: {}
	path: "/xxx"
	query: {}
	[[Prototype]]: Object
}
```

from：从哪里跳转来的信息，是一个对象

```javascript
{
	fullPath: "/xxx"
	hash: ""
	matched: [{…}]
	meta: {}
	name: undefined
	params: {}
	path: "/xxx"
	query: {}
	[[Prototype]]: Object
}
```

next：是一个函数，用来实现跳转功能，调用该函数，就可以实现跳转

```javascript
next()
```

根据to和from中的信息来判断要不要跳转，如果要跳转就调用next()

在路由的配置信息里有一个meta配置项，可以存放一些自定义的数据，叫路由元数据

```javascript
const router = new VueRouter({
	routes: [
		{
			path: '/xxx',
			component: componentName,
			meta: {
				...
			}
		}
	]
})
```

可以在meta中存放一个isAuth属性，值为true，来表示再切换到这个路由组件要进行权限校验

```javascript
meta: {
	isAuth: true
}
```

在beforeEach里就可以这样写：

```javascript
router.beforeEach((to, from, next) => {
	if (to.meta.isAuth) {
		...
	}
})
```

这样的话就省去了冗长的判断是否校验跳转权限过程

#### 全局后置路由守卫：

当**初始化时&路由跳转之后**进行一次操作

```javascript
const router = new VueRouter({ ... })

router.afterEach((to, from) => {
	...
})

export default router
```

to和from参数与全局前置路由守卫的参数一致，但是没有next参数

#### 独享路由守卫

某一个路由所拥有的路由守卫，写在具体的路由配置中，判断该路由组件是否有权限跳转

```javascript
{
	...
	beforeEnter((to, from, next) => {
		...
	})
}
```

名字不再是beforeEach，而是beforeEnter，参数和全局前置一样，使用规则也和全局前置一样

注意：独享路由守卫只有前置，没有后置

#### 组件内路由守卫

写在组件里，而不是路由配置文件中

```javascript
// 在某个组件中
export default {
	...
	beforeRouteEnter(to, from, next) {
		// 当通过路由切换进入之前
	},

	beforeRouteleave(to, from, next) {
		// 当通过路由切换离开之前
	}
}
```

## 36 vue的history和hash工作模式

hash工作模式：

当浏览器地址栏里的url有/#/xxx时，#后面的全是hash值，这些值在发请求时不会带给服务器，例：

url: http://localhost:3000/students/#/aaa/bbb/ccc

服务器接收到的资源请求是：/student/

---

history工作模式：

在路由器配置里使用mode配置项将工作模式切换为history

```javascript
const router = new VueRouter({
	mode: 'history',
	...
})
```

在history下，浏览器地址栏的url没有/#/，所有的url都会带给服务器

在项目上线部署时，浏览器刷新页面时会产生一个404的问题，因为浏览器将前端路由传递给服务器了

解决方法：在服务器端（node.js/express）使用connect-history-api-fallback第三方库解决，这个库可以分辨前端路由和后端api

安装：npm install --save connect-history-api-fallback

导入并使用：

```javascript
var history = require('connect-history-api-fallback')
express.use(history)
```

## 37 Vue UI组件库

### 移动端常用的ui组件库

[Vant](https://youzan.github.io/vant "https://youzan.github.io/vant")

[Cube](https://didi.github.io/cube-ui "https://didi.github.io/cube-ui")

[Mint UI](https://mint-ui.github.io "https://mint-ui.github.io")

### PC端常用ui组件库

[Element UI](https://element.eleme.cn "https://element.eleme.cn")

[IView UI](https://www.iviewui.com "https://www.iviewui.com")

### Element UI组件库的按需引入
