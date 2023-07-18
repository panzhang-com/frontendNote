# Vue3 笔记

## 1. vue-cli 创建的 vue3 项目的结构分析

### 1.1 `main.js` 文件

---

```js
import { createApp } from Vue
import App from './App.vue'

createApp(App).mount('#app')
```

- 引入的不再是 Vue 的构造函数，而是名为 `createApp` 的工厂函数

```js
import {createApp} from Vue
```

- 创建应用实例对象，这个应用实例对象，比之前的 vm 更“轻”

```js
const app = createApp(App)
```

- 挂载容器

```js
app.mount('#app')
```

- `mian.js` 中不兼容 Vue2 写法

### 1.2 `App.vue` 文件

---

```html
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld msg="Welcome to Your Vue.js App" />
</template>
```

- 在组件中可以有多个根标签，这在 vue2 中是不允许的

## 2. 常用 Composition API

### 2.1 setup 配置

---

1. setup 是 vue3 中的一个新的配置项，值为一个函数
2. setup 是所有组合式 api 表演的舞台
3. 组件中所用到的数据，方法等，都要配置到 setup 中
4. setup 函数的两种返回值：

- 若返回一个对象，则对象中的属性，方法在模板中均可直接使用（重点）
- 若返回一个渲染函数，则可以自定义渲染内容（了解）

5. 注意点：

- 尽量不要与 vue2.x 配置混用
  - vue.x 配置（data, methods, computed...）中可以方位到 setup 中的属性，方法
  - setup 中不能访问到 vue2.x 配置
  - 如有重名，setup 优先
- setup 不能是一个 async 函数，因为返回值不再是 return 的对象，而是 promise，模板看不到 return 对象中的属性

### 2.2 ref 函数

---

- 直接写在 setup 中的数据没有响应式

```js
setup() {
  let name = 'zhangsan'
  let age = 18
}
```

- 如果想要将 setup 中的数据配置成响应式的数据（数据一变，页面跟着变），要使用 ref 函数
- 具体用法：

```js
// 1. 先导入 ref 函数
import { ref } from 'vue'

setup() {
  // 2. 将变量的值交给ref函数，该函数返回一个 refImpl 实例对象，对该值进行了加工
  let name = ref('zhangsan')
  let age = ref(18)

  // 3. 修改或者得到数据的值时，要使用 .value 属性，这样一来，vue 就可以检测到数据的变化并重新渲染页面
  name.value = 'lisi'
  age.vale = 20
}
```

- 被 ref 函数加工后的值，将变成一个 refImpl 的实例对象，这是其有响应式的根本原因

```js
setup() {
  let name = 'zhangsan'
  console.log(name)
  /*
    zhangsan
  */

 let name = ref('zhangsan')
 console.log(name)
 /*
  {
    _value: 'zhangsan',
    value: 'zhangsan',

    [[prototype]] {
      value: 'zhangsan',
      get value() {},
      set value() {} 如果数据变化，在这里会对页面重新渲染
    }
  }
 */

}
```

- 当 ref 函数接受的值是一个引用类型（对象）时，内部使用了一个名叫 reactive 的函数来将对象中的属性变成响应式

### 2.3 reactive 函数

---

- 作用：定义一个对象类型的响应式数据（基本类型不能使用，使用 ref 函数）
- 语法：`const proxyObj = reactive(proxyedObj)` 接受一个对象（或数组），返回一个 proxy 的代理对象
- reactive 定义的响应式对象是深层次的
- 内部是基于 ES6 的 proxy 实现的，通过代理对象操作源对象内部数据都是响应式的

### 2.4 vue3 中的响应式原理

---

- 类似于这样的代码：

```js
function reactive(obj) {
  // 遍历 obj 上的所有属性，找到 Object 类型属性，对其进行代理，进行深度监视
  for (let key in obj) {
    if (typeof obj[key] === 'object') {
      obj[key] = reactive(obj[key])
    }
  }

  // 使用 Proxy 进行代理
  return new Proxy(obj, {
    get(target, propKey) {
      return Reflect.get(target, propKey)
    },

    set(target, propKey, value) {
      console.log(`update ${propKey}`)

      // 如果修改的新数据是对象类型，对其进行代理，进行深度监视
      if (typeof value === 'object') {
        Reflect.set(target, propKey, reactive(value))
      } else {
        Reflect.set(target, propKey, value)
      }

      return true
    },

    deleteProperty(target, propKey) {
      console.log(`delete ${propKey}`)
      return Reflect.deleteProperty(target, propKey)
    },
  })
}
```

### 2.5 ref 函数和 reactive 函数对比

---

- 从定义数据角度
  - ref 函数用来定义基本类型
  - reactive 用来定义对象类型数据
  - ref 也可以用来定义对象类型数据，不过底层也使用了 reactive
- 从原理角度
  - ref 使用了 Object.defineProperty()
  - reactive 使用了 Proxy
- 从使用角度
  - ref 定义的数据要使用 `.value` 操作数据，（模板中不用使用）
  - reactive 定义的数据不用使用 `.value`

### 2.6 setup 的两个注意点

---

- setup 的执行时机

  - 在 beforeCreate **之前**执行一次（比 beforeCreate 还要早执行），this 是 undefined

- setup 的参数
  - props：值为对象，包含组件外部传递过来**且组件内部使用 props 配置声明接受了**的属性
  - context：上下文对象
    - attrs：值为对象，包含：组件外部传递过来，但没有在 props 配置中声明接受的属性，相当于 `this.$attrs`
    - slots：收到的插槽内容，相当于 `this.$slots`，vue3 中具名插槽使用时要在 template 标签使用`v-slot:slotName` 才能指定插槽名称
    - emit：分发自定义事件函数，相当于 `this.$emit`，在子组件中要使用 emits 配置接收绑定的自定义事件，否则会报警告，配置项用法和 props 配置一样，使用的时候直接 `context.emit('eventName', ...args)` 即可

### 2.7 计算属性

---

1. 引入 `computed`

```js
import { computed } from 'vue'
```

2. 使用 computed api

```js
setup() {
  const data = reactive({
    a: 1
  })

  // 简写形式 这种方式计算属性不能被修改，是只读的
  let dataComputed = computed(() => {
    return data.a + 1
  })

  // 完整写法 这种方式计算属性可以被修改，修改时会调用 set() 方法
  let dataComputed = computed({
    get() {
      return data.a + 1
    },

    set(value) {
      data.a = value
    }
  })
}
```

### 2.8 监视属性

---

- 先引入 watch 方法

```js
import { watch } from 'vue'
```

#### 2.8.1 监视 ref 定义的基本类型数据

1. 监视一个 ref 定义的数据

```js
setup() {
  const one = ref('first data')
  const two = ref('second data')

  watch(one, (newValue, oldValue) => {
    console.log('watch first data')
  })
}
```

2. 监视多个 ref 定义的数据

```js
setup() {
  const one = ref('first data')
  const two = ref('second data')

  watch([one, two], (newValue, oldValue) => {
    console.log('both watch first and second data')
  })
}
```

当监视多个属性时，`newValue` 和 `oldValue` 分别是这些被监视的属性的新值数组和旧值数组，例如当 `one` 的值被修改为 `'first data was changed'` 时，`newValue` 的值为：`['first data was changed', 'second data']`，`oldValue` 的值为 `['first data', 'second data']`

3. 有其他配置项的监视时（相当于 vue2 中的监视属性的完整写法）

```js
setup() {
  const one = ref('first data')
  const two = ref('second data')

  watch(one, (newValue, oldValue) => {
    console.log('watch first data')
  }, {
    // write other configs up here
    immedata: true
    // ...
  })
}
```

4. 监视 ref 定义的对象类型数据时,要开启深度监视,或者直接监视 value 属性

```js
setup() {
  const person = ref({
    name: 'Alma Dean',
    age: 72
  })

  watch(person, (newValue, oldValue) => {
    console.log('data changed')
  }, {
    deep: true
  })

  // or

  watch(person.value, (newValue, oldValue) => {
    console.log('data changed')
  })
}
```

#### 2.8.2 监视 reactive 定义的对象类型数据

这是一个需要监视的 reactive 定义的数据

```js
setup() {
  const person = reactive({
    name: 'Janie Payne',
    age: 60,

    job: {
      length: 2,

      job1: {
        jobName: 'frontend',
        salary: 30
      },

      job2: {
        jobName: 'backend',
        salary: 30
      }
    }
  })
}
```

1. 监视 person 中的**所有属性**

- 这样监视时的几个注意点
  - 无法正确获取 oldValue 的值,oldValue 和 newValue 是一样的
  - 强制开启深度监视( deep 配置项无效)

```js
watch(person, (newValue, oldValue) => {
  console.log('data was changed')
})
```

2. 监视 person 中的**某个基本类型属性**

- 这样监视时的几个注意点
  - 要监视的属性应该以函数的返回值的方式给出
  - 可以正确获得 oldValue

```js
// watching name in person
watch(
  () => person.name,
  (newValue, oldValue) => {
    console.log('data was changed')
  },
)
```

3. 监视 person 中的**某些基本类型属性**

- 这样监视的几个注意点
  - 要监视的属性应该以函数返回值的形式给出,并且是一个数组
  - 可以正确获得 oldValue
  - newValue 和 oldValue 和监视多个 ref 定义的基本类型数据时是一样的

```js
// watching name and age in person
watch(
  () => [person.name, person.age],
  (newValue, oldValue) => {
    console.log('data was changed')
  },
)
```

4. 监视 person 中的**某个对象类型属性**

- 这样监视的几个注意点
  - 要监视的属性应该以函数返回值的方式给出
  - 不能正确获得 oldValue 的值
  - 这时 deep 配置是有效的

```js
// watching job in person
watch(
  () => person.job,
  (newValue, oldValue) => {
    console.log('data was changed')
  },
  {
    deep: true,
  },
)
```

### 2.9 使用 watchEffect 方法来监视数据

---

- 先引入 watchEffect

```js
import { watchEffect } from 'vue'
```

- 使用 watchEffect

```js
setup() {
  const one = ref(0)
  const person = reactive({
    name: 'Josephine Diaz'
  })

  watchEffect(() => {
    console.log(one.value, person.name)
  })
}
```

- 特点:
  - watchEffect 不像 watch, 不用指明要监视哪个属性, 回调中用到了哪个数据,当这个数据发生变化时,会自动调用一次这个回调
  - 这个回调在数据准备好之后会立即执行一次(相当于开启了 immedate)

## 3. vue3 生命周期

- vue3 中的生命周期钩子在 setup 中通过组合式 api 使用时需要换名

```js
beforeCreate() ===> setup()
created() ===> setup()

// 其余生命周期钩子使用时均需在选项式生命周期钩子的名字前加 on，并采用小驼峰命名，例
beforeMount() ===> onBeforeMount()
mounted() ===> onMounted()
```

- 使用时需要将一个回调以参数的形式传递给生命周期钩子，这点和选项式也不同

```js
onBeforeMount(() => {
  console.log('lift cycle hook executed...')
})
```

## 4. 自定义 hook 函数

- 本质是一个函数，把 setup 中使用的组合式 api 进行封装
- 类似于 vue2 中的 mixin
- 自定义 hook 的优势：复用代码，使 setup 中的逻辑更清晰易懂

## 5. toRef 函数

- 作用；创建一个 ref 对象，其 value 值指向另一个对象中的某个属性
- 语法 `const name = toRef(person, 'name')`
- 要将响应式对象的某个属性单独提供给外部使用时
- 扩展：`toRefs` 与 `toRef` 功能一致，但可以批量创建多个对象，语法：`toRefs(person)`

## 6. shallowRef 和 shallowReactive

- shallowRef：当定义对象类型的数据时，内部不会调用 reactive
  - 使用时机：当明确一个对象类型以后只会被整体替换，而不是改变其中的属性时，可以使用 shallowRef
- shallowReactive；只能将对象的第一层属性定义成响应式
  - 使用时机：当一个对象层次很深，但只要求第一层数据具有响应式时，可以使用 shallowReactive

## 7. readonly 和 shallowReadonly

- readonly：可以将响应式数据变成只读的

```js
let person = reactive({
  name: 'Bradley Perry',
})
person = readonly(person)

// person.name = 'Minerva Foster' [warning] person.name is readonly
```

- shallowReadonly：如果传入的数据是对象类型的，那么只会把第一层的属性变成只读的

- 使用场景：如果一个响应式数据是从其它组件接收到的，并且不希望这个响应式数据被修改，否则会影响其他组件，那么可以使用 readonly 或者 shallowReadonly

## 8. toRaw 和 markRaw

- toRaw：

  - 作用：将一个 **reactive** 生成的响应式对象转为普通对象
  - 使用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面的修改

- markRaw：
  - 作用：标记一个对象，使其永远不会成为响应式对象
  - 使用场景：
    1. 有些值不应该被设置成响应式，例如复杂的第三方类库等
    2. 当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能

## 9. customRef

- 作用：创建一个自定义的 ref，并对其依赖项跟踪和更新触发进行显示的控制
- 实现防抖效果

```js
<template>
  <input type="text" v-model="keyWord" />
  <h3>{{ keyWord }}</h3>
</template>

<script setup>
import { customRef } from 'vue'

// define custom ref function
function showInputRef(value, delay) {
  let timer = null

  return customRef((track, trigger) => {
    return {
      get() {
        track() // inform vue to track the change of value
        return value
      },

      set(newValue) {
        clearTimeout(timer) // Clear the timer before next setting timeout for anti-shake (防抖)

        timer = setTimeout(() => {
          value = newValue
          trigger() // inform vue to reparse template
        }, delay)
      },
    }
  })
}

let keyWord = showInputRef('hello', 500)
</script>
```

- 注意点：
  1. 首先创建一个自定义的 ref 函数，这个自定义的函数必须能够接受一个参数和返回一个使用 `customRef()` 创建的对象

```js
function myRef(value) {
  return customRef()
}
```

2. `customRef()` 函数，接受两个参数，返回一个对象

   - track：这个参数表示让 get 函数可以追踪到 value 的变化
   - trigger：这个参数表示当 value 的值发生变化时让 vue 重新解析模板
   - 返回一个对象：
     - 这个对象必须实现了 `get()` 和 `set(newValue)`，在 get 返回 value 之前调用 `track()`，让 get 追踪 value 的变化，在 set 修改了 value 的值之后调用 `trigger()` 通知 vue 去重新解析模板

```js
function myRef(value) {
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newValue) {
        value = newValue
        trigger()
      },
    }
  })
}
```

## 10. provide 和 inject

- 作用：实现祖组件向后代组件传递数据
- 使用方式：在祖组件中使用 `provide()` 来提供数据，在后代组件中使用 `inject()` 来接受数据
- 写法：

```js
// 祖组件中
provide('dataName', data)

// 后代组件中
const dataName = inject('dataName')
```

- 注意：在祖组件向自己的第一层孩子组件传递数据时，一般不使用这种方式，直接使用父子间传递数据的方式（props）

## 11. 响应式数据的判断：

- 检查一个值是否为 ref 对象：isRef()
- 检查一个值是否为 reactive 创建出来的对象：isReactive()
- 检查一个值是否为 readonly：isReadonly()
- 检查一个值是否为一个由 reactive 或者 readonly 修饰的 proxy 对象：isProxy()

## 12. Composition api 对比 Options api 的优势

1. Options api 存在的问题：

- 新增或者修改一个属性，就需要分别在 data, methods, computed 里面修改

2. Composition api 的优势：

- 可以更加优雅的组织代码，函数，使相关功能的代码集合在一块，提高内聚性

## 13. Teleport 组件

- 作用：是一种能够将组件 html 结构移动到指定位置的技术

```html
<teleport to="html元素选择器">
  <!-- 一些其他html元素 -->
</teleport>
```

- 将 teleport 标签包裹的所有 html 元素移动到 to 属性指定的 html 元素内部的最后，这些 html 元素的样式也使根据移动后所在的元素生效

## 14. Suspense 组件

1. 异步动态引入组件

- 之前引入组件的方式都是静态的引入，这样的缺点是只要子组件没有完全引入，父组件的内容也不能展示

```js
import SonComp from './components/SonComp.vue'
```

- 动态异步引入子组件的方式，可以让子组件在没有完全引入时，先展示父组件的内容，等到子组件完全引入后，再展示子组件内容。这样的缺点是在子组件没有展示时，页面应该提示用户这里还有内容

```js
import { defineAsyncComponent } from 'vue'
const SonComp = defineAsyncComponent(() => import('./components/SonComp.vue'))
```

2. 针对异步动态引入的缺点，可以使用 Suspense 组件解决

```vue
<template>
  <Suspense>
    <template v-slot:default>
      <SonComp />
    </template>
    <template v-slot:fallback>
      <h3>loading...</h3>
    </template>
  </Suspense>
</template>

<script setup>
import { defineAsyncComponent } from 'vue'
const SonComp = defineAsyncComponent(() => import('./components/SonComp.vue'))
</script>
```

- Suspense 组件底层使用具名插槽实现的
- `v-slot:default` 表示要展示的组件，`v-slot:fallback` 表示如果要展示的组件没有加载出来，那么就展示这个内容。插槽名称不能修改

## 15. vue3 中的其它 api 调整

1. vue2 中的 Vue 身上的全局 api 调整到 vue3 的 app 身上

|         全局 api         |          实例 api           |
| :----------------------: | :-------------------------: |
|      Vue.config.xxx      |       app.config.xxx        |
| Vue.config.productionTip |            移除             |
|      Vue.component       |        app.component        |
|      Vue.directive       |        app.directive        |
|        Vue.mixin         |          app.mixin          |
|         Vue.use          |           app.use           |
|      vue.prototype       | app.config.globalProperties |

2. 其他调整

- data 选项始终声明成一个函数
- 过度类名更改：

```css
/* vue2 */
.v-enter,
.v-leave-to {
  /* ... */
}
.v-leave,
.v-enter-to {
  /* ... */
}

/* vue3 */
.v-enter-from,
.v-leave-to {
  /* ... */
}

.v-leave-from,
.v-enter-to {
  /* ... */
}
```

- 移除 keyCode 作为 v-on 的修饰符，同时也不在支持 config.keyCodes
- 移除 v-on.native 修饰符
- 移除过滤器

  - 推荐使用计算属性或方法调用去替换过滤器

- ......

## 16. vue3 中的全局事件总线

1. 安装 mitt 插件

```
npm i --save mitt
```

2. 在一个新的 bus.ts 文件中：

```ts
import mitt from 'mitt'

export default mitt()
```

3. 在需要接收数据的组件中：

```ts
import $bus from 'bus.ts'

onMounted(() => {
  $bus.on('eventName', data => {
    // ...
  })
})
```

3. 在需要传递数据的组件中：

```ts
import $bus from 'bus.ts'

$bus.emit('eventName', data)
```

4. mitt 插件 npm 官网：https://www.npmjs.com/package/mitt

## 17. 自定义组件的 v-model 实现父子组件数据同步

- 原理：给自定义组件上使用 v-model 相当于给自定义组件添加了一个 props 属性，并且绑定了一个自定义事件，子组件通过触发这个自定义事件来修改父组件中对应于这个 props 的响应式数据，数据被修改，页面重新渲染，父子组件的页面都得到更新，实现数据同步
- 下面是一个例子：

```vue
<!-- 在父组件中 -->
<template>
  <div>
    <hr />
    Child1: {{ dataInFather }}<br />
    Child2: page id: {{ pageId }}, page size: {{ pageSize }}

    <!--
      给自定义组件使用 v-model：
      1. 相当于给自定义组件先传递了一个props：:modelValue="data"
      2. 再给自定义组件绑定了一个自定义事件：@update:modelValue="handler"， 这个 handler 不用自己写
    -->
    <child1 v-model="dataInFather" />
    <!--
      1. 相当于给自定义组件传递了两个 props：:pageId="pageId" :pageSize="pageSize"
      2. 相当于给自定组件传递了两个自定义事件：@update:pageId="handler" @update:pageSize="handler"
    -->
    <child2 v-model:pageId="pageId" v-model:pageSize="pageSize" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import Child1 from './Child1.vue';
import Child2 from './Child2.vue';

const dataInFather = ref(1000);

// function handler(dataFromChild) {
//   dataInFather.value = dataFromChild;
// }

const pageId = ref(1);
const pageSize = ref(3);
</script>
```

```vue
<!-- 在子组件2中（子组件1中同理） -->
<template>
  <div class="child2">
    <h2>In Child Component 2</h2>
    <button @click="update">update data</button>
    <hr>
    page id: {{ pageId }}, page size: {{ pageSize }}
  </div>
</template>

<script setup lang="ts">
const props = defineProps(['pageId', 'pageSize']);
const $emit = defineEmits(['update:pageId', 'update:pageSize']);

function update() {
  $emit('update:pageId', props.pageId + 1);
  $emit('update:pageSize', props.pageSize + 3);
}
</script>
```