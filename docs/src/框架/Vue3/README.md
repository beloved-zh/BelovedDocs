# 

# 准备

参考资料：

- Vue：https://v3.cn.vuejs.org/
- TypeScript 教程：https://www.runoob.com/typescript/ts-tutorial.html
- TypeScript 中文网：https://www.tslang.cn/index.html
- Vue Cli：https://cli.vuejs.org/zh/
- Vite：https://v3.cn.vuejs.org/guide/installation.html#vite

注意：

- Vue3 的 Vue Devtools 与 Vue2 的不兼容，需要安装新的Vue Devtools

# 创建项目

## Vue-Cli

官网文档：https://cli.vuejs.org/zh/

> 全局安装
>
> - 最新的 vue/cli  官方推荐 node 版本 V10 以上。实测 V16 以上最好

```sh
# 安装
npm install -g @vue/cli

# 查看版本
vue --version
```

> 创建

```sh
vue create <project-name>
```

> 选择一个模板
>
> -  Default ([Vue 3] babel, eslint)：Vue3 + babel + eslint
> -  Default ([Vue 2] babel, eslint)：Vue2 + babel + eslint
> -  Manually select features：自定义

![image-20220407204842157](image/image-20220407204842157.png)

> 选择需要安装的模块包。* ：代表选中

![image-20220407205128682](image/image-20220407205128682.png)

> 选择 Vue 版本

![image-20220407205242376](image/image-20220407205242376.png)

后续步骤根据实际需求进行选择。

其余配置参考官方文档：https://cli.vuejs.org/zh/config/

## Vite

具体可参考官方文档：https://cn.vitejs.dev/guide/

> Vite 需要 [Node.js](https://nodejs.org/en/) 版本 >= 12.0.0。

```sh
npm create vite@latest
# or
yarn create vite
```

根据提示输入项目名，选择模板等操作。

![image-20220410161516194](image/image-20220410161516194.png)

`vite.config.ts`

```typescript
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
// @ts-ignore
import path from "path";

export default defineConfig({
  plugins: [vue()],
  base: "./", // 类似publicPath，'./'避免打包访问后空白页面，要加上，不然线上也访问不了
  resolve: {
    alias: {
      // 如果报错__dirname找不到，需要安装node,执行 npm install @types/node --save-dev 或 yarn add @types/node --save-dev
      "@": path.resolve(__dirname, "src"),
      "@assets": path.resolve(__dirname, "src/assets"),
      "@components": path.resolve(__dirname, "src/components"),
      "@images": path.resolve(__dirname, "src/assets/images"),
      "@views": path.resolve(__dirname, "src/views"),
      "@store": path.resolve(__dirname, "src/store"),
    },
  },
  build: {
    outDir: "dist",
    assetsDir: "assets", //指定静态资源存放路径
    sourcemap: false, //是否构建source map 文件
    terserOptions: {
      // 生产环境移除console
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
  server: {
    https: false, // 是否开启 https
    open: false, // 是否自动在浏览器打开
    port: 3000, // 端口号
    host: "0.0.0.0",
    proxy: {
      "/api": {
        target: "", // 后台接口
        changeOrigin: true,
        secure: false, // 如果是https接口，需要配置这个参数
        // ws: true, //websocket支持
        rewrite: (path) => path.replace(/^\/api/, ""),
      },
    },
  },
  // 引入第三方的配置
  optimizeDeps: {
    include: [],
  },
});
```

# Vue3 新特性

- 组件不在需要唯一根标签，可以存在多个根标签
- 组合式 API 使用前都需要导入

## 生命周期

> Vue3 新增了 `beforeUnmount` 和 `unmounted` 。其实就是对应 Vue2 的 `beforeDestroy` 和 `destroyed`

![实例的生命周期](image/lifecycle.svg)

## v-model

> `v-model` 其实是一个语法糖。通过 `props` 和 `emit` 组合而成的。
>
> 新特性：
>
> - prop：`value` -> `modelValue`；
> - 事件：`input` -> `update:modelValue`；
> - `v-bind` 的 `.sync` 修饰符和组件的 `model` 选项已移除
> - 新增 支持多个 `v-model`
> - 新增 支持自定义 修饰符。通过 `modelModifiers` prop 提供给组件

`App.vue`

```vue
<template>
  <h1>App.vue</h1>
  showChild：{{showChild}}<br><br>
  title：{{title}}<br><br>
  <button @click="showChild = !showChild">切换Chile</button>
  <!-- 多个 v-model 可使用：指定 -->
  <Child v-model.test="showChild" v-model:title.test="title"></Child>
</template>

<script setup lang="ts">
  import Child from './Child.vue'
  import {ref} from 'vue'

  let showChild = ref(true)
  let title = ref('提示标题111')
</script>
```

`Child.vue`

```vue
<template>
  <div v-if="modelValue">
    <h1 >Child.vue</h1>
    title：{{ title }} <br><br>
    <button @click="close">关闭</button> <br><br>
    <button @click="updateTitle">修改标题</button>
  </div>
</template>

<script setup lang="ts">

  // prop 默认值为 modelValue
  type Props = {
    modelValue: boolean,
    modelModifiers: {
      default: () => {}
    },
    title: string,
    // 自定义修饰符
    titleModifiers: {
      prefix: boolean
    }
  }
  const propsData = defineProps<Props>()

  // 双向绑定默认事件为 update:modelValue  自定义的为 update：xxx
  const emit = defineEmits(['update:modelValue', 'update:title'])

  let close = () => {
    console.log(propsData.modelModifiers)
    emit('update:modelValue', false)
  }

  let updateTitle = () => {
    // 查看有指定名称的属性是否有指定的修饰符。后续可根据实际业务做操作
    console.log(propsData.titleModifiers)
    console.log(propsData.titleModifiers?.prefix)
    emit('update:title', '修改后的标题')
  }
</script>
```

## directives

> `directives` 自定义指令。
>
> 官网资料：
>
> - https://v3.cn.vuejs.org/guide/custom-directive.html#%E7%AE%80%E4%BB%8B
> - https://v3.cn.vuejs.org/api/application-api.html#directive
>
> 指令钩子函数：
>
> - `created`：在绑定元素的 attribute 或事件监听器被应用之前调用
> - `beforeMount`：当指令第一次绑定到元素并且在挂载父组件之前调用
> - `mounted`：在绑定元素的父组件被挂载前调用
> - `beforeUpdate`：在更新包含组件的 VNode 之前调用
> - `updated`：在包含组件的 VNode 及其子组件的 VNode 更新后调用
> - `beforeUnmount`：在卸载绑定元素的父组件之前调用
> - `unmounted`：当指令与元素解除绑定且父组件已卸载时，只调用一次
>
> 钩子参数详解：
>
> - `el`：当前绑定的 DOM 元素
> - `binding`：
>   - `instance`：使用指令的组件实例。
>   - `value`：传递给指令的值。例如，在 `v-focus="{background: 'red'}"` 中，该值为 `{background: 'red'}`。
>   - `oldValue`：先前的值，仅在 `beforeUpdate` 和 `updated` 中可用。无论值是否有更改都可用。
>   - `arg`：传递给指令的参数(如果有的话)。例如在 `v-focus:beloved` 中，arg 为 `"beloved"`。
>   - `modifiers`：包含修饰符(如果有的话) 的对象。例如在 `v-focus.input` 中，修饰符对象为 `{input: true}`。
>   - `dir`：一个对象，在注册指令时作为参数传递。钩子函数
> - `vnode`：当前元素的虚拟DOM
> - `prevVnode`：上一个虚拟节点，仅在 `beforeUpdate` 和 `updated` 钩子中可用
>
> 函数简写：
>
> - `mounted` 和 `updated` 时触发相同行为。
> - 不关心其他的钩子函数。
>
> 注意：
>
> - 自定义指令必须是 `v 开头驼峰命名`

```vue
<template>
  <h1>App.vue</h1>
  <div v-if="showInput">
    name：<input type="text" v-model="name" v-focus:beloved.input="{background: 'red'}" v-test="{width: '1000px',height: '100px'}"> <br><br>
  </div>
  <button @click="showInput = !showInput">切换显示</button>
</template>

<script setup lang="ts">
  import {Directive, DirectiveBinding, ref, RendererNode, VNode} from 'vue'

  // 自定义指令必须是 v开头驼峰命名
  const vFocus:Directive = {
    created() {
      console.log('在绑定元素的 attribute 或事件监听器被应用之前调用')
    },
    beforeMount() {
      console.log('当指令第一次绑定到元素并且在挂载父组件之前调用')
    },
    // el：当前绑定的DOM 元素
    // binding：指令相关数据
    // vnode：当前元素的虚拟DOM
    // prevVnode：上一个虚拟节点，仅在 beforeUpdate 和 updated 钩子中可用
    mounted(el:HTMLElement, binding:DirectiveBinding, vnode:RendererNode, prevVnode:any) {
    // mounted(...par) {
      console.log('在绑定元素的父组件被挂载前调用')
      // console.log(par)
      console.log(el)
      console.log(binding)
      console.log(vnode)
      console.log(prevVnode)

      el.style.background = binding.value.background
      if (binding.modifiers.input) {
        el.focus()
      }
    },
    beforeUpdate() {
      console.log('在更新包含组件的 VNode 之前调用')
    },
    updated(el:HTMLElement, binding:DirectiveBinding, vnode:RendererNode, prevVnode:VNode) {
      console.log('在包含组件的 VNode 及其子组件的 VNode 更新后调用')
      console.log(el)
      console.log(binding)
      console.log(vnode)
      console.log(prevVnode)
    },
    beforeUnmount() {
      console.log('在卸载绑定元素的父组件之前调用')
    },
    unmounted() {
      console.log('当指令与元素解除绑定且父组件已卸载时，只调用一次')
    }
  }

  // 函数简写
  // mounted 和 updated 时触发相同行为，不关心其他的钩子函数。
  const vTest:Directive = (el, binding: DirectiveBinding) => {
    el.style.width = binding.value.width
    el.style.height = binding.value.height
  }

  let name = ref()
  let showInput = ref(true)
</script>

```

## hooks

> 自定义Hook。主要用来处理复用代码逻辑的一些封装。相当于 vue2 中的 `Mixins`。
>
> Vue2 中 mixins 的缺点：
>
> - 涉及到覆盖的问题，组件的 data、methods、filters 会覆盖 mixins 里的同名 data、methods、filters。
> - 变量来源不明确（隐式传入），不利于阅读，使代码变得难以维护。
>
> Vue3 自定义 hook ：
>
> - Vue3 的 hook 函数，相当于 vue2 的 mixin, 不同在与 hooks 是函数
> - Vue3 的 hook 函数，可以帮助提高代码的复用性，能在不同的组件中都利用 hooks 函数
>
> hook 库：https://vueuse.org/

自定义 hooks 

```typescript
import {onMounted} from 'vue'

type Options = {
    el: string
}

export default function (options:Options):Promise<string> {
    return new Promise((resolve) => {
        onMounted(() => {
            let img:HTMLImageElement = document.querySelector(options.el) as HTMLImageElement
            console.log('========================')
            console.log(img)
            // 图片加载完毕调用转换 base64
            img.onload = () => {
                resolve(base64(img))
            }
        })

        const base64 = (el:HTMLImageElement) => {
            const canvas = document.createElement('canvas')
            canvas.width = el.width
            canvas.height = el.height
            const ctx = canvas.getContext('2d')
            ctx?.drawImage(el, 0, 0, canvas.width, canvas.height)
            return canvas.toDataURL('image/png');
        }
    })
}
```

使用

```vue
<template>
  <h1>App.vue</h1>
  <div style="height: 300px;display: flex;align-content: center">
    <img src="@assets/logo.png" id="img">
    <textarea style="width: 100%" readonly v-model="base64"></textarea>
  </div>
  <br><br>
</template>

<script setup lang="ts">
import {ref} from 'vue'
// 引入自定义 hook，命名规则一般是 use 开头
import useBase64 from './hooks'


let base64 = ref()

// 调用自定义 hook 函数
useBase64({
  el: '#img'
}).then(res => {
  base64.value = res
})

</script>
```

## 全局函数和变量

> Vue3 没有 `Prototype` 属性 使用 `app.config.globalProperties` 代替定义变量和函数。
>
> - vue2：`Vue.prototype.$http = () => {}`
> - vue3：`createApp({}).config.globalProperties.$http = () => {}`

`main.ts`

```typescript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

type Filter = {
    format: <T extends any>(str: T) => T
}

// 扩展 globalProperties 声明 
declare module 'vue' {
    export interface ComponentCustomProperties {
        $filters: Filter,
        $userName: string
    }
}

// 定义全局方法、变量
app.config.globalProperties.$filters = {
    format<T extends any>(str: T): string {
        return `${str}---${str}`
    }
}
app.config.globalProperties.$userName = 'Beloved'

app.mount('#app')
```

`App.vue`

```vue
<template>
  <h1>App.vue</h1>
  <!-- 模板中可直接读取 -->
  $userName：{{$userName}}<br><br>
  $filters：{{$filters.format('张三')}}<br><br>
</template>

<script setup lang="ts">

// setup 读取
import { getCurrentInstance, ComponentInternalInstance } from 'vue'

const { appContext } = <ComponentInternalInstance>getCurrentInstance()

console.log(appContext.config.globalProperties.$userName)
console.log(appContext.config.globalProperties.$filters.format('李四'))

</script>
```

# Composition API

`Composition API（组合式 API）` Vue3 新引入的概念，Vue2 中虽然是使用 (`data`、`computed`、`methods`、`watch`) 组件选项来组织逻辑进行开发，但有一个大型组件其中会涉及很多逻辑点，不便于维护开发，使用组合式 API 可将相关代码进行收集拆分组件。

官网介绍：https://v3.cn.vuejs.org/guide/composition-api-introduction.html#%E4%BB%80%E4%B9%88%E6%98%AF%E7%BB%84%E5%90%88%E5%BC%8F-api

> 注意：
>
> - 组合式 API 使用前都需要导入

## setup

> setup：
>
> - 可以使用 Vue3 提供的组合式API 的地方，只在初始化时执行一次
> - 函数如果返回对象, 对象中的属性或方法, 模板中可以直接使用
> - 模板中使用变量或者方法都需要返回

```vue
<template>
  {{number}}
</template>

<script lang="ts">
// defineComponent 函数，定义一个组件
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'App',
  setup () {
    let number = 10
    return {
      number
    }
  }
});
</script>
```

### 生命周期

> Vue3 中的生命周期钩子除了和 Vue2 选项式API 一样的写法外，还可以在 setup 中使用组合式API 定义生命周期钩子。
>
> - setup 执行是在 `beforeCreate` 之前执行，此时组件还没有创建
> - setup 中不能使用 `this` ，执行时组件没有创建 `this = undefined` 
> - 生命周期钩子前面加上 `on` 来访问组件的生命周期钩子 
> - `beforeCreate`、`created` 这两个没有对应的组合式 API 的生命周期钩子，其实可以吧 `setup ` 看作对应的钩子。
> - 组合式 API 生命周期钩子函数比选项式的执行时间靠前

| 选项式 API        | Hook inside `setup` |
| ----------------- | ------------------- |
| `beforeCreate`    | Not needed*         |
| `created`         | Not needed*         |
| `beforeMount`     | `onBeforeMount`     |
| `mounted`         | `onMounted`         |
| `beforeUpdate`    | `onBeforeUpdate`    |
| `updated`         | `onUpdated`         |
| `beforeUnmount`   | `onBeforeUnmount`   |
| `unmounted`       | `onUnmounted`       |
| `errorCaptured`   | `onErrorCaptured`   |
| `renderTracked`   | `onRenderTracked`   |
| `renderTriggered` | `onRenderTriggered` |
| `activated`       | `onActivated`       |
| `deactivated`     | `onDeactivated`     |

![image-20220408124625917](image/image-20220408124625917.png)

```vue
<template>
  <h2>setup</h2>
  {{number}} <br><br>
  <button @click="number++">修改数据</button>
</template>

<script lang="ts">
import {
  defineComponent,
  onBeforeMount,
  onBeforeUnmount,
  onBeforeUpdate,
  onMounted, onUnmounted,
  onUpdated,
  reactive,
  ref
} from 'vue';

// 生命周期钩子
export default defineComponent({
  name: 'App',
  setup () {
    // 在组件创建前执行
    console.log('setup')
    // 没有this
    console.log('this==>', this)

    let number = ref(10)

    // 生命周期钩子前面加上 `on` 来访问组件的生命周期钩子
    onBeforeMount(() => {
      console.log('onBeforeMount')
    })

    onMounted(() => {
      console.log('onMounted')
    })

    onBeforeUpdate(() => {
      console.log('onBeforeUpdate')
    })

    onUpdated(() => {
      console.log('onUpdated')
    })

    onBeforeUnmount(() => {
      console.log('onBeforeUnmount')
    })

    onUnmounted(() => {
      console.log('onUnmounted')
    })

    
    return {
      number
    }
  },
  beforeCreate() {
    console.log('beforeCreate')
  },
  created() {
    console.log('created')
  },
  beforeMount() {
    console.log('beforeMount')
  },
  mounted() {
    console.log('mounted')
  },
  beforeUpdate() {
    console.log('beforeUpdate')
  },
  updated() {
    console.log('updated')
  },
  beforeUnmount() {
    console.log('beforeUnmount')
  },
  unmounted() {
    console.log('unmounted')
  }
});
</script>
```

### 返回值

> - 一般返回一个对象，其中包含属性和方法，供模板使用
> - setup 中返回的属性和方法会和 Vue2 中选项式 API 中的属性和方法合并，同名的话  setup 中的优先级高
> - 选项式 API 中可以访问 setup 返回的属性和方法。setup 中不能访问选项式 API 中的属性和方法
> - setup不能是一个async函数：因为返回值不再是return的对象，而是promise，模板看不到return对象中的属性数据

```vue
<template>
  <h2>setup</h2>
  number：{{number}} <br><br>
  name：{{name}} <br><br>
  age：{{age}} <br><br>
  <button @click="update">修改数据</button>
  <button @click="update1">修改数据1</button>
</template>

<script lang="ts">
import {
  defineComponent,
  onBeforeMount,
  onBeforeUnmount,
  onBeforeUpdate,
  onMounted, onUnmounted,
  onUpdated,
  reactive,
  ref
} from 'vue';

export default defineComponent({
  name: 'App',
  data () {
    return {
      number: 5,
      name: '张三'
    }
  },
  setup () {

    let number = ref(3)

    let age = ref(10)

    let update = () => {
      number.value++

      // setup 中不能访问 选项式 API 的属性或方法
      // name += '!'
    }

    return {
      number,
      age,
      update
    }
  },
  methods: {
    update1 () {
      // 选项式 API 可以访问 setup 返回的属性和方法
      this.number++
      this.name += "!"
      this.age++
      this.update()
    }
  }
});
</script>
```

### 参数

> - `props`：接收父组件传递子组件的值，并且是使用 props 接收的参数。是响应式的数据，不能用 ES6 解构
> - `context`：普通 JavaScript 对象，暴露了其它可能在 `setup` 中有用的值，不是响应式的可以使用 ES6 解构
>   - `attrs`：接收父组件传递子组件的值，并且没有在 props 接收的参数。相当于 `this.$attrs`
>   - `emit`：用来分发自定义事件的函数。相当于 `this.$emit`
>   - `expose`：该函数允许通过公共组件实例暴露特定的 property。
>   - `slots`：包含所有传入的插槽内容的对象。 相当于 `this.$slots`

`App.vue`

```vue
<template>
  <h2>父组件</h2>
  number：{{number}}<br><br>
  <button @click="number++">更新数据</button>
  <hr>
  <Child :number="number" test="test" @emitUpdate="emitUpdate"></Child>
</template>

<script lang="ts">
import {
  defineComponent,
  ref
} from 'vue';
import Child from "@/components/Child.vue";

export default defineComponent({
  name: 'App',
  components: {
    Child
  },
  setup () {

    let number = ref(10)

    let emitUpdate = (val:number) => {
      console.log(val)
      number.value += val
    }

    return {
      number,
      emitUpdate
    }
  }
});
</script>
```

`Child.vue`

```
<template>
  <h2>子组件</h2>
  number：{{number}}<br><br>
  <button @click="updateNumber">更新数据</button>
</template>

<script>
export default {
  name: "Child",
  props: ['number'],
  // setup (props, context) {
  //   console.log("context：", context)
  setup (props, {attrs, emit, expose, slots}) {
    console.log('Child ----- setup ----- init')
    console.log("poops：", props)
    console.log("attrs：", attrs);
    console.log("emit：", emit);
    console.log("expose：", expose);
    console.log("slots：", slots);

    let updateNumber = () => {
      emit('emitUpdate', 10)
    }

    return {
      updateNumber
    }
  }
}
</script>
```

### script setup

参考官方文档：https://v3.cn.vuejs.org/api/sfc-script-setup.html#%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95

`<script setup>`： 是在单文件组件 (SFC) 中使用组合式 API 的编译时语法糖。相比于普通的 `<script>` 语法，它具有更多优势：

- 更少的样板内容，更简洁的代码。
- 能够使用纯 Typescript 声明 props 和抛出事件。
- 更好的运行时性能 (其模板会被编译成与其同一作用域的渲染函数，没有任何的中间代理)。
- 更好的 IDE 类型推断性能 (减少语言服务器从代码中抽离类型的工作)。

> 就是 `setup () {}` 的语法糖，简单写法，具体可参考官网。

```vue
<template>
  number：{{ number }} <br><br>
  <button @click="number++">number++</button>
</template>

<script setup lang="ts">
  import {ref} from "vue";

  const number = ref(10)
</script>
```

## refs

`ref`：接收一个值返回响应式可变 Ref 对象。ref 对象仅有一个 `.value` property，指向该内部值。

- 一般用来定义一个基本类型的响应式数据

> 直接定义属性不是响应式数据，操作数据改变，页面不同步渲染

![image-20220410172505504](image/image-20220410172505504.png)

```vue
<template>
  number：{{ number }} <br><br>
  <button @click="numberAdd">更新数据</button>
</template>

<script setup lang="ts">
 
  let number:number = 10

  const numberAdd = () => {
    number++
    console.log(number);
  }

</script>
```

> 定义响应式数据
>
> - js 中操作数据：`xxx.value`
> - 模板中操作数据：不需要 `.value`

![image-20220410173051485](image/image-20220410173051485.png)

```vue
<template>
  number：{{ number }} <br><br>
  <button @click="numberAdd">更新数据</button>
</template>

<script setup lang="ts">
  import { Ref, ref } from 'vue'

  let number = ref<number>(10)

  // let number:Ref<number> = ref(10)

  const numberAdd = () => {
    number.value++
    console.log(number)
  }

</script>
```

### isRef

> 判断是不是一个 ref 对象

![image-20220410173830396](image/image-20220410173830396.png)

```vue
<script setup lang="ts">
  import { isRef, Ref, ref } from 'vue'

  let number = ref<number>(10)
  
  let msg = 'hello'

  console.log('number：', isRef(number))
  console.log('msg：', isRef(msg))
  
</script>
```

### shallowRef

> 创建一个跟踪自身 `.value` 变化的 ref，但不会使其值也变成响应式的。

```vue
<template>
  user{{ user }} <br><br>
  <button @click="update">更新数据</button>
</template>

<script setup lang="ts">
  import { shallowRef } from 'vue'

  let user = shallowRef({
    name: '张三'
  })

  const update = () => {
    // 其 .value 是响应式 但内部的值不是响应式
    // user.value.name = '李四'     // 不是响应式修改
    user.value = {                  // 响应式
        name: '李四'
    }
    console.log(user)
  }

</script>
```

### triggerRef

> 强制更新页面DOM。可以与 shallowRef 配合使用

```vue
<template>
  user{{ user }} <br><br>
  <button @click="update">更新数据</button>
</template>

<script setup lang="ts">
  import { shallowRef, triggerRef  } from 'vue'

  let user = shallowRef({
    name: '张三'
  })

  
  const update = () => {
    
    // 非响应式
    user.value.name = '李四'

    console.log(user)

    // 强制更新 DOM
    triggerRef(user)
  }

</script>
```

### customRef

> - `customRef` 用于自定义返回一个 ref 对象，可以显式地控制依赖追踪和触发响应，接受工厂函数
> - 两个参数分别是用于追踪的 `track` 与用于触发响应的 `trigger`，并返回一个带有 `get` 和 `set` 属性的对象
> - 通过 `customRef` 返回的 ref 对象，和正常 ref 对象一样，通过 `.value` 修改或读取值

```vue
<template>
  message：<input type="text" v-model="message"><br><br>
  {{message}}
</template>

<script setup lang="ts">
  import { customRef } from 'vue'

  // 自定义防抖 Ref
  function MyRef<T>(value:string, delay = 500) {
    let timeout:any
    return customRef((track, trigger) => {
      return {
        get() {
          track()  // 追踪当前数据
          return value 
        },
        set(newValue:string) {
          clearTimeout(timeout)
          timeout = setTimeout(() => {
            value = newValue
            trigger()  // 触发响应,即更新界面
          }, delay)
        }
      }
    })
  }

  let message = MyRef<string>('')

</script>
```

### toRef

> 可以用来为源响应式对象上的某个 property 新创建一个 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref)。然后，ref 可以被传递，它会保持对其源 property 的响应式连接。
>
> 从源响应式对象上的某个 property 结构一个 ref

```vue
<template>
  Object：{{user}}<br><br>
  wife：{{wife}}<br><br>
  <button @click="update">更新数据</button>
</template>

<script setup lang="ts">
  import { reactive, toRef } from 'vue'

  let user = reactive({
    name: '张三',
    wife: {
      name: '小红'
    }
  })

  // 从源响应式对象上的某个 property 结构一个 ref
  let wife = toRef(user, 'wife')

  const update = () => {

    // 修改结构对象也会改变原始对象
    wife.value.name += '!'

    console.log('wife', wife);
    console.log('user：', user);  
  }

</script>

```

### toRefs

> 将响应式对象转换为普通对象，其中结果对象的每个 property 都是指向原始对象相应 property 的 [`ref`](https://v3.cn.vuejs.org/api/refs-api.html#ref)。
>
> 将源响应式对象上的所有 property 结构成 ref

```vue
<template>
  Object：{{user}}<br><br>
  name：{{name}}<br><br>
  name：{{wife}}<br><br>
  <button @click="update">更新数据</button>
</template>

<script setup lang="ts">
  import { reactive, toRefs } from 'vue'

  let user = reactive({
    name: '张三',
    wife: {
      name: '小红'
    }
  })

  // 将源响应式对象上的所有 property 结构成 ref
  let {name, wife} = toRefs(user)

  const update = () => {

    // 修改结构对象也会改变原始对象
    name.value += '1'
    wife.value.name += '!'

    console.log('name', name);
    console.log('wife', wife);
    console.log('user：', user);  
  }

</script>
```



## reactive

> reactive：
>
> - 定义一个响应式对象属性，返回一个 Proxy 代理对象
> - 不能定义基本数据类型
> - 响应式转换是“深层的”：会影响对象内部所有嵌套的属性

![image-20220410185740491](image/image-20220410185740491.png)

```vue
<template>
  Object：{{user}}<br><br>
  Array：{{nameList}}<br><br>
  <button @click="update">更新数据</button>
</template>

<script setup lang="ts">
  import { reactive } from 'vue'

  // reactive 不能定义基本数据类似
  // let number = reactive(10)

  let user = reactive({
    name: 'zhngsan',
    age: 20
  })

  let nameList = reactive(['张三', '李四'])

  const update = () => {
    // 直接赋值会破坏响应式
    // user = {
    //   name: '李四',
    //   age: 22
    // }

    // nameList = ['王五']

    // reactive 属性操作不需要 .value
    user.name = '李四'
    user.age = 22
    nameList.push('王五')

    console.log('user：', user)
    console.log('nameList', nameList)
  }

</script>
```

### readonly

> 接受一个对象 (响应式或纯对象) 或 [ref](https://v3.cn.vuejs.org/api/refs-api.html#ref) 并返回原始对象的只读代理。只读代理是深层的：任何被访问的嵌套 property 也是只读的。

![image-20220410190214648](image/image-20220410190214648.png)

![image-20220410190229367](image/image-20220410190229367.png)

```vue
<template>
  Object：{{user}}<br><br>
  <button @click="update">更新数据</button>
</template>

<script setup lang="ts">
  import { reactive, readonly } from 'vue'

  let user = reactive({
    name: 'zhngsan',
    age: 20
  })

  // 拷贝一份proxy对象将其设置为只读
  let copyUser = readonly(user)

  const update = () => {
    user.age = 22

    // 只读属性不能被修改
    copyUser.age =30
  }

</script>
```

### isProxy

> 检查对象是否是由 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 或 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 创建的 proxy。

### isReactive

> 检查对象是否是由 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 创建的响应式代理。

### isReadonly

> 检查对象是否是由 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 创建的只读代理。

### shallowReactive

> 创建一个响应式代理，它跟踪其自身 property 的响应性，但不执行嵌套对象的深层响应式转换 (暴露原始值)。
>
> 一个只能对浅层的数据修改 如果是深层的数据只会改变值 不会改变视图
>
> 注意：
>
> - 页面 `DOM` 挂载之前的修改是有效的
> - 同时修改自身和深层数据也是有效的，修改自身属性完毕之后重新渲染 DOM 也会将深层数据渲染

```vue
<template>
  Object：{{user}}<br><br>
  <button @click="update1">一块修改</button>
  <button @click="update2">修改自身</button>
  <button @click="update3">修改嵌套对象</button>
</template>

<script setup lang="ts">
  import { shallowReactive } from 'vue'

  // 定义一个只能对浅层的数据修改 如果是深层的数据只会改变值 不会改变视图
  let user = shallowReactive({
    name: '张三',
    age: 20,
    wife: {
      name: '小红',
      age: 20
    }
  })

  // 注意：页面DOM挂载之前的修改是有效的
  user.age++
  user.wife.age++

  // 同时修改也是有效的，修改自身属性完毕之后重新渲染 DOM 也会将深层数据渲染
  const update1 = () => {
    user.age++
    user.wife.age++
  }

  // 自身属性是响应式的，是有效的
  const update2 = () => {
    user.age++
  }

  // 深层数据非响应式 修改只是数据变化，但页面不渲染
  const update3 = () => {
    user.wife.age++
  }

</script>
```

### toRaw

> 返回 [`reactive`](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 或 [`readonly`](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly) 代理的原始对象。这是一个“逃生舱”，可用于临时读取数据而无需承担代理访问/跟踪的开销，也可用于写入数据而避免触发更改。**不**建议保留对原始对象的持久引用。请谨慎使用。

![image-20220410211908389](image/image-20220410211908389.png)

```vue
<template>
</template>

<script setup lang="ts">
  import { reactive, toRaw, isReactive, isProxy } from 'vue'

  let user = reactive({
    name: '张三',
    wife: {
      name: '小红'
    }
  })
  // 将响应式对象转为原始对象
  let copyUser = toRaw(user)

  console.log(user)
  console.log(isProxy(user))
  console.log(isReactive(user))

  console.log(copyUser)
  console.log(isProxy(copyUser))
  console.log(isReactive(copyUser))
  
</script>
```

## Vue2 与 Vue3 的响应式

### Vue2

> - 对象：通过 defineProperty 对象的已有属性值的读取和修改进行劫持（ 监视 / 拦截 ）
> - 数组：通过重写数组更新数组一系列更新元素的方法来实现元素修改的劫持
> - 对象添加未定义属性或删除已有属性不是响应式，得通过 `Vue.set()` 操作
> - 直接通过下标替换元素或更新length，界面不会自动更新

```js
Object.defineProperty(data, 'count', {
    get () {}, 
    set () {}
})
```

### Vue3 

> - 通过Proxy(代理): 拦截对data任意属性的任意(13种)操作, 包括属性值的读写, 属性的添加, 属性的删除等...
> - 通过 Reflect(反射): 动态对被代理对象的相应属性进行特定的操作
> - 文档:
>   - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
>   - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect

```js
new Proxy(data, {
	// 拦截读取属性值
    get (target, prop) {
    	return Reflect.get(target, prop)
    },
    // 拦截设置属性值或添加新属性
    set (target, prop, value) {
    	return Reflect.set(target, prop, value)
    },
    // 拦截删除属性
    deleteProperty (target, prop) {
    	return Reflect.deleteProperty(target, prop)
    }
})
```

## computed

> - 简写：只接收一个 `getter` 函数，返回一个处理过后的响应式对象
> - 完全写法：接收一个 `get` 和 `set` 函数的对象，可以创建一个可写的响应式对象

```vue
<template>
  原始数据：{{user}}<br><br>
  nameAge：{{nameAge}}<br><br>
  number：{{number}}<br><br>
  <input type="text" v-model="numberCom">
</template>

<script setup lang="ts">
  import { computed, reactive, ref } from 'vue'

  let user = reactive({
    name: '张三',
    age: '20'
  })

  let number = ref(10)

  // 接受一个 getter 函数，并根据 getter 的返回值返回一个不可变的响应式 ref 对象
  let nameAge = computed(() => {
    return `${user.name}---${user.age}`
  })
  
  // 接受一个具有 get 和 set 函数的对象，用来创建可写的 ref 对象。
  let numberCom = computed({
    get: () => {
      return number.value + 10
    },
    set: (newValue) => {
      number.value = newValue - 10
    }
  })

</script>
```

## watch

> `watch`：组合式 API 与选项式 API 完全等效。`watch` 需要侦听特定的数据源，并在单独的回调函数中执行副作用。
>
> 注意：
>
> - 监听单个值
>   - 参数1：需要监听的值
>   - 参数2：监听到的回调函数，接收新值和旧值
>   - 参数3：配置对象
>     - deep：开启深度监听
>     - immediate：鉴听开始之后被立即调用
> - 监听多个值
>   - 参数1：传入数组，数组内的是需要监听的值
>   - 监听回调的参数也是数组格式，内容为监听数据集合的新旧值 
> - 监听 ref 对象
>   - 正常情况 ref 是监听不到的，需要开启深度监听，才可以监听到
>   - 对象的监听 newValue 和 oldValue 的值是一样的
> - 监听 reactive 对象
>   - 无论是否开启深度监听，都可以监听到
>   - 对象的监听 newValue 和 oldValue 的值是一样的
> - 监听对象单一属性
>   - 参数1：是一个函数，返回需要监听对象的属性

```vue
<template>
  number1：<input type="text" v-model="number1"><br><br>
  number2：<input type="text" v-model="number2"><br><br>
  refObject：<input type="text" v-model="refObject.user.name"><br><br>
  reactiveObject：<input type="text" v-model="reactiveObject.user.name"><br><br>
  reactiveObjectAge：<input type="text" v-model="reactiveObject.user.age"><br><br>
</template>

<script setup lang="ts">
  import { ref, reactive, watch } from 'vue'

  let number1 = ref()
  let number2 = ref()

  // 监听单个值
  //    参数1：需要监听的值
  //    参数2：监听到的回调函数，接收新值和旧值
  //    参数3：配置对象
  //        deep：开启深度监听
  //        immediate：鉴听开始之后被立即调用
  watch(number1, (newValue, oldValue) => {
    console.log('newValue：', newValue)
    console.log('oldValue：', oldValue)
  }, {
    deep: true,
    immediate: true
  })

  // 监听多个值
  //    1：传入数组，数组内的是需要监听的值
  //    2：监听回调的参数也是数组格式，内容为监听数据集合的新旧值 
  watch([number1, number2], (newValue, oldValue) => {
    console.log('newValue：', newValue)
    console.log('oldValue：', oldValue)
  })

  let refObject = ref({
    user: {
      name: '张三'
    }
  })

  // 监听 ref 对象。正常情况 ref 是监听不到的，需要开启深度监听，才可以监听到
  //   注意：对象的监听 newValue 和 oldValue 的值是一样的
  watch(refObject, (newValue, oldValue) => {
    console.log('newValue：', newValue)
    console.log('oldValue：', oldValue)
  }, {
    deep: true
  })

  let reactiveObject = reactive({
    user: {
      name: '李四',
      age: 20
    }
  })

  // 监听 reactive 对象。无论是否开启深度监听，都可以监听到
  //   注意：对象的监听 newValue 和 oldValue 的值是一样的
  watch(reactiveObject, (newValue, oldValue) => {
    console.log('newValue：', newValue)
    console.log('oldValue：', oldValue)
  })

  // 监听对象单一属性
  //    参数1：是一个函数，返回需要监听对象的属性
  watch(() => {
    return reactiveObject.user.age
  }, (newValue, oldValue) => {
    console.log('newValue：', newValue)
    console.log('oldValue：', oldValue)
  })
</script>
```

### watchEffect

> 立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。只要作用域中用到的属性发生变化就会回调。
>
> - 参数1：传入一个函数，函数的参数（onInvalidate）是一个回调函数。会在调用整体代码块之前调用。
> - 参数2：其余配置项
>   - 副作用刷新时机 flush。默认是 pre 一般使用 post
>   - onTrack 和 onTrigger 选项可用于调试侦听器的行为。只能在开发模式下工作
> - 返回值也是一个回调函数，可通过返回值停止监听。停止监听时也会调一次 onInvalidate

```vue
<template>
  name：<input type="text" v-model="name"><br><br>
  age：<input type="text" v-model="age"><br><br>
  money：<input type="text" v-model="money"><br><br>
  <!-- 调用 watchEffect 的返回值可以停止监听 -->
  <button @click="watchEffectResult()">停止监听</button><br><br>
</template>

<script setup lang="ts">
  import { ref, reactive, watch, watchEffect } from 'vue'

  let name = ref()
  let age = ref()
  let money = ref()

  // 立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。只要作用域中用到的属性发生变化就会回调。
  //    传入一个函数，函数的参数是一个回调函数。会在调用整体代码块之前调用
  //    返回值也是一个回调函数，可通过返回值停止监听。停止监听时也会调一次 onInvalidate
  const watchEffectResult =  watchEffect(onInvalidate => {
    console.log('name：', name.value)
    console.log('age：', age.value)
    // 会在调用整体代码块之前调用
    onInvalidate(() => {
      console.log('watchEffectBefer')
    })
  },{
    //flush: 'pre'    // 默认  组件更新前执行
    //flush: 'sync'   // 强制效果始终同步触发
    flush: 'sync',   // 组件更新后执行
    // onTrack 和 onTrigger 选项可用于调试侦听器的行为。只能在开发模式下工作
    onTrack () {  // 将在响应式 property 或 ref 作为依赖项被追踪时被调用。

    },
    onTrigger () {  // 将在依赖项变更导致副作用被触发时被调用。

    }
  })

 
</script>
```

## props

> 选项式写法参考：[官网链接](https://v3.cn.vuejs.org/api/options-data.html#props) 。setup 函数写法参考：[官网链接](https://v3.cn.vuejs.org/guide/composition-api-setup.html#props) 或 [本文](/src/框架/Vue3/README?id=参数) 。此处展示 `<script setup>` 写法。
>
> - 通过 `defineProps` 函数接收父组件传递的 props 。参数基本与 选项式相同
> - `defineProps` 只能在 `<script setup>` 中使用，所以不需要导入.
> - 返回的是 Proxy 的代理对象，模板中使用 props 的值可以直接使用，js 中使用需要接收返回值
> - `TS` 写法参数定义是在类型中
> - `TS` 中可以通过 `withDefaults` 定义默认值
>   - 参数1：props 函数
>   - 参数2：一个对象设置默认值

`App.vue`

```vue
<template>
  <h1>父组件</h1>
  <Child :name="name" :age="22"></Child>
</template>

<script setup lang="ts">
  // <script setup> 中引用组件之后，可以直接在模板中使用
  import Child from './Child.vue'

  import { ref } from 'vue'

  let name = ref('张三')
  let age = ref(20)

</script>
```

`Child.vue`

```vue
<template>
  <h2>子组件</h2>
  name：{{name}} <br><br>
  age：{{age}} <br><br>
  hobby：{{hobby}} <br><br>
</template>

<script setup lang="ts">

  // 通过 defineProps 函数接收父组件传递的 props 。参数基本与 选项式相同
  // defineProps 只能在 <script setup> 中使用，所以不需要导入
  // 返回的是 Proxy 的代理对象，模板中使用 props 的值可以直接使用，js 中使用需要接收返回值

  // 数组写法
  // let props = defineProps(['name', 'age'])
  // console.log(props)
  // console.log(props.name)

  // 对象写法 可以声明类型、默认值
  // defineProps({
  //   name: String,
  //   age: {
  //     type: Number,
  //     default: 20,
  //   },
  //   hobby: {
  //     type: Array,
  //     default: () => {
  //       return ['抽烟', '喝酒', '烫头']
  //     }
  //   }
  // })

  // TS 写法， 基本写法
  // defineProps<{
  //   name: string,
  //   age: number,
  //   hobby: string[]
  // }>()

  // TS 默认值写法
  // 通过 withDefaults 函数
  //    第一个参数是 props 函数
  //    第二个参数是一个对象设置默认值
  type Props = {
    name?: string,
    age?: number,
    hobby?: string[]
  }
  withDefaults(defineProps<Props>(), {
    age: 20,
    hobby: () => ['抽烟', '喝酒']
  })
</script>
```

## emits

> 选项式写法参考：[官网链接](https://v3.cn.vuejs.org/api/options-data.html#emits) 。setup 函数写法参考：[官网链接](https://v3.cn.vuejs.org/guide/composition-api-setup.html#context) 或 [本文](/src/框架/Vue3/README?id=参数) 。此处展示 `<script setup>` 写法。
>
> - 通过 `defineEmits` 定义自定义函数，参数是自定义函数名数组，通过返回值可触发
> - `TS` 专属可定义函数参数、返回值类型

`App.vue`

```vue
<template>
  <h1>父组件</h1>
  number：{{number}}<br><br>
  <Child @on-click="getNumber"></Child>
</template>

<script setup lang="ts">
  import Child from './Child.vue'

  import { ref } from 'vue'

  let number = ref()

  const getNumber = (val: number) => {
    number.value = val
  }

</script>
```

`Child.vue`

```vue
<template>
  <h2>子组件</h2>
  number：{{number}} <br><br>
  <button @click="addNumber">点击增加传递给父组件</button>
</template>

<script setup lang="ts">

  import {Ref, ref} from 'vue'

  let number = ref<number>(10)

  // defineEmits 定义自定义事件
  // const emit = defineEmits(['on-click'])

  // TS 专门写法定义函数参数类型和返回值
  const emit = defineEmits<{
    (e: 'on-click', number: number): void
  }>()

  const addNumber = () => {
    number.value++
    // 调用
    emit('on-click', number.value)
  }
</script>
```

## 模板 ref

> - 响应式 API 用法 [官网说明](https://v3.cn.vuejs.org/guide/component-template-refs.html#%E6%A8%A1%E6%9D%BF%E5%BC%95%E7%94%A8)
> - 组合式 API 中：模板 ref 引用需要定义一个 `ref` 对象进行使用 [官网说明](https://v3.cn.vuejs.org/guide/composition-api-template-refs.html#%E6%A8%A1%E6%9D%BF%E5%BC%95%E7%94%A8)
> - `<script setup>` 中需要在子组件通过`defineExpose` 暴露出父组件可以通过 `ref` 访问的对象或方法

`App.vue`

```vue
<template>
  <h1>父组件</h1>
  number：{{number}}<br><br>
  <button @click="getNumber">点击获取子组件数据</button><br><br>
  <button @click="onChildFun">点击调用子组件方法</button><br><br>
  <Child ref="child"></Child>
</template>

<script setup lang="ts">
  import Child from './Child.vue'

  import { ref } from 'vue'

  // 父组件需要定义一个 ref 对象接收子组件实例
  // 名称需要和 ref 名称一致
  let child = ref()

  let number = ref()

  const getNumber = () => {
    number.value = child.value.number
  }

  const onChildFun = () => {
    child.value.addNumber()
  }

</script>
```

`Child.vue`

```vue
<template>
  <h2>子组件</h2>
  number：{{number}} <br><br>
</template>

<script setup lang="ts">

  import { ref } from 'vue'

  let number = ref<number>(10)

  const addNumber = () => {
    console.log('Child#addNumber()调用了');
    
    number.value++
  }

  // 通过 defineExpose 暴露父组件可以通过 ref 访问的属性或方法
  defineExpose({
    number,
    addNumber
  })
</script>
```

## Provide / Inject

> 使用 `provide` 和 `inject` ，无论组件层次结构有多深，父组件都可以作为其所有子组件的依赖提供者。
>
> 父组件使用 `provide` 选项来提供数据，子组件使用 `inject` 选项来接收使用数据。
>
> `provide`：暴露需要提供的数据。如果希望使用者不能修改可以使用 `readonly `
>
> - 参数1 是key值
> - 参数2 是value
>
> `inject`：
>
> - 参数1 是需要接收的key
> - 参数2 是默认值
>
> 参数是响应式 任何一层进行修改会通知到所有使用者

![image-20220414211003558](image/image-20220414211003558.png)

`App.vue`

```vue
<template>
  <h1>App.vue</h1>
  number：{{number}}<br><br>
  <button @click="addNumber">number++</button><br><br>
  <Child></Child>
</template>

<script setup lang="ts">
  import Child from './Child.vue'

  import { ref, provide  } from 'vue'

  let number = ref(10)

  // 参数1 是key值
  // 参数2 是value
  provide('number', number)

  const addNumber = () => {
    number.value++
  }

</script>
```

`Child.vue`

```vue
<template>
  <hr/>
  <h2>Child.vue</h2>
  number：{{number}} <br><br>
  <button @click="addNumber">number++</button><br><br>
  <Sun></Sun>
</template>

<script setup lang="ts">
  import Sun from './Sun.vue'
  import { inject, ref } from 'vue'

  let number = inject('number', ref(15))

  const addNumber = () => {
    number.value++
  }
</script>
```

`Sun.vue`

```vue
<template>
  <hr/>  
  <h3>Sun.vue</h3>
  number：{{number}} <br><br>
  <button @click="addNumber">number++</button><br><br>
</template>

<script setup lang="ts">
    
  import { inject, ref } from 'vue'

  // 参数1 是需要接收的key
  // 参数2 是默认值
  let number = inject('number', ref(20))

  // 参数是响应式 任何一层进行修改会通知到所有使用者
  const addNumber = () => {
    number.value++
  }
inject
  
</script>
```

# Mitt

Vue3 移除了 `$on`、`$off`、`$once` 等自带的自定义事件相关方法，vue3 中推荐使用 `mitt` 库来使用事件总线传递数据。

GitHub：https://github.com/developit/mitt

## 安装

```sh
npm i mitt -S
# or
yarn add mitt
```

## 挂载全局 mitt

`main.ts`

```typescript
import { createApp } from 'vue'
import App from './App.vue'
// 引入 mitt
import mitt from 'mitt'

const Mitt = mitt()

const app = createApp(App)

// 扩展 globalProperties 声明 
declare module 'vue' {
    export interface ComponentCustomProperties {
        $EventBus: typeof Mitt
    }
}

// 挂载全局实例，要挂载在config.globalProperties上
app.config.globalProperties.$EventBus = Mitt

app.mount('#app')
```

## 触发自定义事件

> `mitt` 挂载在了全局，在 `setup` 中需要使用 `getCurrentInstance` 获取组件实例，才可以使用
>
> `$EventBus.emit(eventType,params)`

```vue
<template>
  <h2>A.vue</h2>
  number：{{number}}<br><br>
  <button @click="cli">传递number</button>
</template>

<script setup lang="ts">
  import { ref, getCurrentInstance } from 'vue'

  // 获取组件实例
  const instance = getCurrentInstance()

  let number = ref(10)

  const cli = () => {
    // 触发自定义事件
    instance?.proxy?.$EventBus.emit('on-push', number)

    instance?.proxy?.$EventBus.emit('push-msg', 'message')
  }
</script>
```

## 注册并监听自定义事件

> `$EventBus.on(eventType,callback)`
>
> 注意：第一个参数如果为 `*` 代表监听所有事件触发。此时，callback 回调函数有 2 个参数，1 是事件类型，2 是实际参数

```vue
<template>
  <h2>B.vue</h2>
  number：{{number}}<br><br>
</template>

<script setup lang="ts">
  import { ref, getCurrentInstance } from 'vue'

  // 获取组件实例
  const instance = getCurrentInstance()

  let number = ref()

  // 注册并监听自定义事件
  //    参数1：事件名
  //    参数2：回调函数 参数为接收值
  instance?.proxy?.$EventBus.on('on-push', (val) => {
    console.log('val：', val)
    number.value = val
  })

  // 注册并监听所有自定义事件
  //    参数1：*
  //    参数2：回调函数 第一个参数是事件名，第二个参数是接收值
//   instance?.proxy?.$EventBus.on('*', (type, val) => {
//       console.log('type：', type)
//       console.log('val：', val)
//   })
</script>
```

## 事件取消

> - 取消指定事件：
>   - `$EventBus.off(eventType,callback)`
>   - 需要将回调定义在外部
> - 取消全部事件：
>   - `$EventBus.all.clear()`

```vue
<template>
  <h2>B.vue</h2>
  number：{{number}}<br><br>
</template>

<script setup lang="ts">
  import { ref, getCurrentInstance } from 'vue'

  // 获取组件实例
  const instance = getCurrentInstance()

  let number = ref()

  const fun = (val: any) => {
    console.log('val：', val)
    number.value = val
  }


  instance?.proxy?.$EventBus.on('on-push', fun)
  
  // 取消指定事件的监听。注意：需要将回调定义在外部
//   instance?.proxy?.$EventBus.off('on-push', fun)

  // 取消所有事件监听
  instance?.proxy?.$EventBus.all.clear()
</script>
```

# Pinia

## 介绍

github：https://github.com/vuejs/pinia

官方文档：https://pinia.vuejs.org/

优点：

- 支持 TS
- 轻量，压缩后的体积只有1kb左右
- 去除 mutations，只有 state，getters，actions
- actions 支持同步和异步
- 代码扁平化没有模块嵌套，只有 store 的概念，store 之间可以自由使用，每一个store都是独立的
- 无需手动添加 store，store 一旦创建便会自动添加

## 安装

```shell
yarn add pinia
# or
npm install pinia
```

## 注册

### vue2

```javascript
import { createPinia, PiniaVuePlugin } from 'pinia'
 
Vue.use(PiniaVuePlugin)
const pinia = createPinia()
 
new Vue({
  el: '#app',
  // other options...
  // ...
  // note the same `pinia` instance can be used across multiple Vue apps on
  // the same page
  pinia,
})
```

### vue3

```typescript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// pinia
import { createPinia } from 'pinia'
const store = createPinia()
app.use(store)

app.mount('#app')
```

## 初始化

```typescript
import { defineStore } from 'pinia'

// 定义存储库（hook 一般使用 use 开头）
// 参数：
//   id：存储的唯一id，pinia 使用它连接 devtools
export const useTestStore = defineStore('TEST', {
    state: () => {
        return {
            age: 1,
            username: '张三'
        }
    },
    // 等效于 计算属性
    getters: {
        
    },
    // 等效于 方法，可以做同步异步
    actions: {
        
    }
})
```

`App.vue`

```vue
<template>
  <h1>App.vue</h1>
  age：{{store.age}}<br><br>
  username：{{store.username}}
</template>

<script setup lang="ts">
  import { ref } from 'vue'
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
</script>

<style scoped>

</style>
```

![image-20220516102821210](image/image-20220516102821210.png)

## state

`/store/index.ts`

```typescript
import { defineStore } from 'pinia'

// 定义存储库（hook 一般使用 use 开头）
// 参数：
//   id：存储的唯一id，pinia 使用它连接 devtools
export const useTestStore = defineStore('TEST', {
    state: () => {
        return {
            age: 1,
            username: '张三'
        }
    },
    // 等效于 计算属性
    getters: {
        
    },
    // 等效于 方法，可以做同步异步
    actions: {
        addAge (num:number) {
            this.age += num
        }
    }
})
```

> state 修改方式：
>
> - 直接修改 `store.age++`
> - 通过 `$patch` 修改，传递一个对象，可以有多个值，对象中没有的则不会修改
> - 通过 `$patch` 修改，传递一个函数，可以在修改时做逻辑处理，参数是 `store` 的 `state`
> - 通过 `$state` 修改，必须覆盖整个 `state` 对象
> - 通过 `action`

```vue
<template>
  <h1>App.vue</h1>
  age：{{store.age}} <button @click="ageAdd">age++</button><br><br>
  username：{{store.username}}
</template>

<script setup lang="ts">
  import { ref } from 'vue'
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
  
  const countAdd = () => {
    // 1：直接修改
    //  store.age++
    
    // 2：通过 $patch 修改，传递一个对象，可以有多个，对象中没有的则不会修改
    // store.$patch({
    //   age: 888,
    //   username: '李四'
    // })
    
    // 3：通过 $patch 修改，传递一个函数，可以在修改时做逻辑处理，参数是 store 的 state
    // store.$patch((state) => {
    //   console.log('state', state)
    //   if (state.age > 0) {
    //     state.age--
    //   } else {
    //     state.age++
    //   }
    // })

    // 4：通过 $state 修改，必须覆盖整个 state 对象
    // store.$state = {
    //   age: 10,
    //   username: '李四'
    // }
    
    // 5：通过 action 
    store.addAge(10)
  }
</script>

<style scoped>

</style>
```

> pinia 的 state 解构：
>
> - 直接结构会失去响应式
> - 使用 `storeToRefs` 结构具有响应式效果

```vue
<template>
  <h1>App.vue</h1>
  count：{{store.age}} <button @click="ageAdd">age++</button><br><br>
  username：{{store.username}} <br><br>
  <hr>
  <h3>storeToRefs解构</h3>
  count：{{age}} <button @click="countAge2">age++</button><br><br>
  username：{{username}} <br><br><hr>
</template>

<script setup lang="ts">
  import { ref } from 'vue'
  import { useTestStore } from './store/index'
  import {storeToRefs} from 'pinia'
  
  const store = useTestStore()
  
  // pinia 直接解构失去响应式
  // const { age, username } = store
  
  const { age, username } = storeToRefs(store)
  
  const ageAdd = () => {
    store.addAge(10)
  }
  
  const ageAdd2 = () => {
    age.value++
  }
</script>

<style scoped>

</style>
```

## getters

> 相当于计算属性：
>
> - 普通函数 可以使用 `this`
> - 使用箭头函数后 不能使用 `this`，需要使用参数传递的参数 `state`

`index.ts`

```typescript
import { defineStore } from 'pinia'

// 定义存储库（hook 一般使用 use 开头）
// 参数：
//   id：存储的唯一id，pinia 使用它连接 devtools
export const useTestStore = defineStore('TEST', {
    state: () => {
        return {
            age: 1,
            username: '张三'
        }
    },
    // 等效于 计算属性
    getters: {
        // 使用箭头函数后 不能使用 this，需要使用 state
        nameAge: (state) => {
            return `${state.username}---${state.age}`
        },
        // 普通函数 可以使用 this
        sex(): string {
          return this.age % 2 === 0 ? '男' : '女'  
        },
        // 互相调用
        nameAgeSex() {
          return `${this.nameAge}---${this.sex}`  
        }
    },
    // 等效于 方法，可以做同步异步
    actions: {
        addAge (num:number) {
            this.age += num
        }
    }
})
```

`App.vue`

```vue
<template>
  <hr>
  <h2>state：</h2>
  age：{{store.age}} <button @click="ageAdd">age++</button><br><br>
  username：{{store.username}} <br><br>
  <hr>
  <h2>getters：</h2>
  nameAge：{{store.nameAge}}<br><br>
  sex：{{store.sex}}<br><br>
  nameAgeSex：{{store.nameAgeSex}}<br><br>
</template>

<script setup lang="ts">
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
  
  const ageAdd = () => {
    store.age++
  }
  
</script>

<style scoped>

</style>
```

## actions

> 等效于 方法，可以做同步异步提交：
>
> - 同步 直接调用即可
> - 异步 结合async await 修饰

`index.ts`

```typescript
import { defineStore } from 'pinia'

type User = {
    username: string,
    age: number
}

const Login = (): Promise<User> => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve({
            username: '张三',
            age: 25
        })
    }, 1000)  
  })
}

// 定义存储库（hook 一般使用 use 开头）
// 参数：
//   id：存储的唯一id，pinia 使用它连接 devtools
export const useTestStore = defineStore('TEST', {
    state: () => {
        return {
            age: 1,
            username: '张三',
            user: <User>{}
        }
    },
    // 等效于 计算属性
    getters: {
        // 使用箭头函数后 不能使用 this，需要使用 state
        nameAge: (state) => {
            return `${state.username}---${state.age}`
        },
        // 普通函数 可以使用 this
        sex(): string {
          return this.age % 2 === 0 ? '男' : '女'  
        },
        // 互相调用
        nameAgeSex() {
          return `${this.nameAge}---${this.sex}`  
        }
    },
    // 等效于 方法，可以做同步异步
    actions: {
        addAge (num:number) {
            this.age += num
        },
        setUser () {
            this.user = {
                username: '李四',
                age: 20
            }
        },
        async Login() {
            const data = await Login()
            this.user = data
        }
    }
})
```

`App.vue`

```vue
<template>
  <hr>
  <h2>actions：</h2>
  username：{{store.user.username}}<br><br>
  age：{{store.user.age}}<br><br>
  <button @click="setUser">同步setUser</button>
  <button @click="Login">异步Login</button>
</template>

<script setup lang="ts">
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
  
  const setUser = () => {
    store.setUser()
  }
  
  const Login = () => {
    store.Login()
  }
</script>

<style scoped>

</style>
```

## API

### $reset

> https://pinia.vuejs.org/api/interfaces/pinia._StoreWithState.html#reset
>
> 重置 `state` 为初始状态 

```vue
<template>
  <hr>
  <h2>actions：</h2>
  username：{{ store.user.username }}<br><br>
  age：{{ store.user.age }}<br><br>
  sex：{{ store.sex }}<br><br>
  <button @click="Login">Login</button>
  <button @click="reset">重置</button>
</template>

<script setup lang="ts">
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
  
  const Login = () => {
    store.Login()
  }

  const reset = () => {
    // 重置 state
    store.$reset()
  }
</script>

<style scoped>

</style>
```

### $onAction

> https://pinia.vuejs.org/api/interfaces/pinia._StoreWithState.html#onaction
>
> 每次调用 `Action` 的监听回调
>
> - 参数1：`callback` 回调函数
>   - `store`：调用的 store
>   - `name`：action 名称
>   - `args`：action 接收的参数
>   - `after()`：action 调用成功后回调
>   - `onError()`：action 调用失败回调函数
> - 参数2：组件卸载之后当前监听是否有效
> - 返回值是个函数，可删除当前设置

```vue
<template>
  <hr>
  <h2>actions：</h2>
  username：{{ store.user.username }}<br><br>
  age：{{ store.user.age }}<br><br>
  sex：{{ store.sex }}<br><br>
  <button @click="Login">Login</button>
  <button @click="reset">重置</button>
</template>

<script setup lang="ts">
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
  
  const Login = () => {
    store.Login()
  }

  const reset = () => {
    // 重置 state
    store.$reset()
  }
  
  // 每次调用 Action 的回调
  //   参数1：调用前回调函数
  //   参数2：组件卸载是是否销毁当前监听是否有效
  store.$onAction((args) => {
    // 调用的 store
    console.log(args.store)
    // action 名称
    console.log(args.name)
    // action 接收的参数
    console.log(args.args)
    // action 调用成功后回调
    args.after((resolvedValue) => {
      console.log(resolvedValue)
    })
    // action 调用失败回调函数
    args.onError((error) => {
      console.error(error)
    })
  }, false)
</script>

<style scoped>

</style>
```

### $subscribe

> https://pinia.vuejs.org/api/interfaces/pinia._StoreWithState.html#subscribe
>
> `state` 的监听事件
>
> - 参数1：`callback` 回调函数
>   - `args`： store 等数据，修改前后
>   - `state`：state 响应式数据对象
> - 参数2：
>   - `detached`：组件卸载之后当前监听是否有效
>   - `options` 配置对象，和监听事件的配置一样
> - 返回值是个函数，可删除当前设置

```vue
<template>
  <hr>
  <h2>actions：</h2>
  username：{{ store.user.username }}<br><br>
  age：{{ store.user.age }}<br><br>
  sex：{{ store.sex }}<br><br>
  <button @click="Login">Login</button>
  <button @click="reset">重置</button>
</template>

<script setup lang="ts">
  import { useTestStore } from './store/index'
  
  const store = useTestStore()
  
  const Login = () => {
    store.Login()
  }
  
  // state 的监听事件
  store.$subscribe((args,state) => {
    // store 等数据，修改前后
    console.log(args)
    // state 响应式数据对象
    console.log(state)
  }, {
    detached: false
  })
</script>

<style scoped>

</style>
```

## 持久化插件

`piniaPlugin.ts`

```typescript
import {PiniaPluginContext} from 'pinia'
import {toRaw} from 'vue'

const setStorage = (key: string, val: any) => {
  localStorage.setItem(key, JSON.stringify(val))
}

const getStorage = (key: string) => {
    return localStorage.getItem(key) ? JSON.parse(localStorage.getItem(key) as string) : {}
}

type Options = {
    key?: string
}

const DEFAULT_KEY = 'app'

const piniaPlugin = (options?: Options) => {
    return (context: PiniaPluginContext) => {
        console.log('context', context)
        const { store } = context
        const data = getStorage(`${options?.key ?? DEFAULT_KEY}-${store.$id}`)
        store.$subscribe(() => {
            setStorage(`${options?.key ?? DEFAULT_KEY}-${store.$id}`, toRaw(store.$state))
        })
        return {
            ...data
        }
    }
}

export {
    piniaPlugin
}
```

`main.ts`

```typescript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// pinia
import {createPinia} from 'pinia'
import { piniaPlugin } from './store/piniaPlugin'
const store = createPinia()
store.use(piniaPlugin())

app.use(store)


app.mount('#app')
```

# Router

> github：https://github.com/vuejs/router
>
> 官方文档：https://router.vuejs.org/zh/

## 安装

> 注意：
>
> - Vue3  对应 router4 版本
> - Vue2  对应 router3 版本

```shell
npm install vue-router@4
# or
yarn add vue-router@4
```

## 注册配置

`@router/index.ts`

```typescript
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

const routes: Array<RouteRecordRaw> = [{
    path: '/',
    name: 'Main',
    component: () => import('@views/main.vue')
},{
    path: '/setup',
    name: 'Setup',
    component: () => import('@views/setup.vue')
}]


// history  =>  createWebHistory
// hash     =>  createWebHashHistory  
// abstact  =>  createMemoryHistory
const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出router
export default router
```

`mian.ts`

```typescript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

import router from './router'
app.use(router)

app.mount('#app')
```

`App.vue`

```vue
<template>
  <h1>App.vue</h1>
  <!-- 路由跳转 -->
  <router-link to="/">main</router-link>&emsp;
  <router-link to="/setup">setup</router-link>
  <hr>
  <!-- 路由渲染 -->
  <router-view></router-view>
</template>

<script setup lang="ts">

</script>

<style scoped>

</style>

```

## 路由跳转

> - `to="路由地址"`
> - `to="{name: '路由名称'}"`
> - `router.push('路由地址')`
> - `router.push({path: '路由地址'})`
> - `router.push({name: '路由名称'})`

```vue
<template>
  <h1>App.vue</h1>
  <!-- 路由跳转 -->
  <span>path跳转：</span>
  <router-link to="/">main</router-link>&emsp;
  <router-link to="/setup">setup</router-link><br><br>
  <span>name跳转：</span>
  <router-link :to="{name: 'Main'}">main</router-link>&emsp;
  <router-link :to="{name: 'Setup'}">setup</router-link><br><br>
  <span>编程式导航path：</span>
  <button @click="toPagePath('/')">main</button>&emsp;
  <button @click="toPagePath('/setup')">setup</button><br><br>
  <span>编程式导航name：</span>
  <button @click="toPageName('Main')">main</button>&emsp;
  <button @click="toPageName('Setup')">setup</button><br><br>
  <hr>
  <!-- 路由渲染 -->
  <router-view></router-view>
</template>

<script setup lang="ts">

  import { useRouter } from 'vue-router'
  
  const router = useRouter()

  const toPagePath = (url: string) => {
    // 字符串
    // router.push(url)
    router.push({
      path: url
    })
  }

  const toPageName = (name: string) => {
    router.push({
      name: name
    })
  }

</script>

<style scoped>

</style>
```

## 历史记录

> - `replace` 可以不保存路由跳转的历史记录
>   - `<router-link replace to="/">main</router-link>`
>   - `router.replace(url)`
> - `router.go(1)`：表示在历史堆栈中前进或后退多少步，正数前进，负数后退
> - `router.back()`：后退一步

```vue
<template>
  <h1>App.vue</h1>
  <span>没有历史记录：</span>
  <router-link replace to="/">main</router-link>&emsp;
  <router-link replace to="/setup">setup</router-link><br><br>
  <span>有历史记录：</span>
  <router-link to="/">main</router-link>&emsp;
  <router-link to="/setup">setup</router-link><br><br>
  <span>没有历史记录：</span>
  <button @click="toReplace('/')">main</button>&emsp;
  <button @click="toReplace('/setup')">setup</button><br><br>
  <span>有历史记录：</span>
  <button @click="toPush('/')">main</button>&emsp;
  <button @click="toPush('/setup')">setup</button><br><br>
  <!-- 前进后退 -->
  <button @click="prev()">后退</button>&emsp;
  <button @click="next()">前进</button><br><br>
  <hr>
  <!-- 路由渲染 -->
  <router-view></router-view>
</template>

<script setup lang="ts">

  import { useRouter } from 'vue-router'
  
  const router = useRouter()

  const toPush = (url: string) => {
    router.push(url)
  }

  const toReplace = (url: string) => {
    router.replace(url)
  }

  const next = () => {
    router.go(1)
  }
  
  const prev = () => {
    // router.go(-1)
    router.back()
  }

</script>

<style scoped>

</style>
```

## 路由传参

> - 路由路径跳转时，需要使用 `query` 进行传参
>   - `query` 传参配置的是 `path`
>   - `query`传参和 `get` 请求一致，参数在 `url` 后是拼接的
> - `query` 传参使用 `useRoute`的`query`进行接收
> - 路由名称跳转时，需要使用 `params` 进行传参
>   - `params` 传参配置的是 `name`
>   - `params` 传参时 地址栏没有参数
> - `params` 传参使用 `useRoute`的`params`进行接收
> - `params`传参刷新会无效，但是 `query` 会保存传递过来的值，刷新不变 
> - 动态路由传参：*路径参数* 用冒号 `:` 表示。
>   - 例如：`/setup/:id/:userName/:sex/:age`
>   - 参数使用 `restful` 风格路径参数在地址后，刷新不会消失

`index.ts`

```typescript
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

const routes: Array<RouteRecordRaw> = [{
    path: '/',
    name: 'Main',
    component: () => import('@views/main.vue')
},{
    path: '/setup-query',
    name: 'SetupQuery',
    component: () => import('@views/setup.vue')
},{
    path: '/setup/:id/:userName/:sex/:age',
    name: 'Setup',
    component: () => import('@views/setup.vue')
}]


// history  =>  createWebHistory
// hash     =>  createWebHashHistory  
// abstact  =>  createMemoryHistory
const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出router
export default router
```

`App.vue`

```vue
<template>
  <h1>App.vue</h1>
  <hr>
  <router-view></router-view>
</template>

<script setup lang="ts">
</script>

<style scoped>
</style>
```

`main.vue`

```vue
<template>
  <el-table :data="data" style="width: 100%">
    <el-table-column prop="userName" label="姓名" />
    <el-table-column prop="sex" label="性别" />
    <el-table-column prop="age" label="年龄" />
    <el-table-column label="操作" >
      <template #default="scope">
        <el-button type="primary" @click="toQuery(scope.row)">Query传参</el-button>
        <el-button type="primary" @click="toParams(scope.row)">Params传参</el-button>
      </template>
    </el-table-column>
  </el-table>
</template>

<script setup lang="ts">

import { data } from '@data/userList.json'
import { useRouter } from 'vue-router'

type User = {
  id: string;
  userName: string;
  sex: string;
  age: number;
}

const router = useRouter()

// query 传参正常get 地址栏显示参数
const toQuery = (user: User) => {
  
  router.push({
    path: "/setup-query",
    query: user
  })
  
}

// params 传参地址栏不显示，接收页面刷新会丢失数据
// 如果使用动态路由参数，参数使用 restful 风格路径参数在地址后，刷新不会消失
const toParams = (user: User) => {
  
  router.push({
    name: 'Setup',
    params: user
  })
  
}
</script>

<style scoped>

</style>
```

`setup.vue`

```vue
<template>
  <el-row :gutter="10">
    <el-col :span="12">
      <el-card class="box-card ">
        <template #header>
          <div class="card-header">
            <span>用户信息--query</span>
            <el-button class="button" text @click="router.back()">返回</el-button>
          </div>
        </template>
        <div class="text item">id：{{route.query.id}}</div>
        <div class="text item">用户民：{{route.query.userName}}</div>
        <div class="text item">性别：{{route.query.sex}}</div>
        <div class="text item">年龄：{{route.query.age}}</div>
      </el-card>
    </el-col>
    <el-col :span="12">
      <el-card class="box-card">
        <template #header>
          <div class="card-header">
            <span>用户信息--params</span>
            <el-button class="button" text @click="router.back()">返回</el-button>
          </div>
        </template>
        <div class="text item">id：{{route.params.id}}</div>
        <div class="text item">用户民：{{route.params.userName}}</div>
        <div class="text item">性别：{{route.params.sex}}</div>
        <div class="text item">年龄：{{route.params.age}}</div>
      </el-card>
    </el-col>
  </el-row>
</template>

<script setup lang="ts">

  import { useRoute, useRouter } from 'vue-router'
  
  const route = useRoute()

  const router = useRouter()
  
</script>

<style scoped>
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .text {
    font-size: 14px;
  }
  
  .item {
    margin-bottom: 18px;
  }
  
</style>
```

## 路由嵌套

> - 父路由的 `path` 不为空，跳转子路由时需要写全路径
> - 子路由 `path` 为空，则此路由为父路由的默认子路由

`index.ts`

```typescript
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

const routes: Array<RouteRecordRaw> = [{
    path: '/',
    name: 'Main',
    component: () => import('@views/main.vue'),
    children: [
        {
            path: '/',
            name: 'childrenA',
            component: () => import('@views/childrenA.vue')
        },
        {
            path: '/childrenB',
            name: 'childrenB',
            component: () => import('@views/childrenB.vue')
        }
    ]
},{
    path: '/setup-query',
    name: 'SetupQuery',
    component: () => import('@views/setup.vue')
},{
    path: '/setup/:id/:userName/:sex/:age',
    name: 'Setup',
    component: () => import('@views/setup.vue')
}]


// history  =>  createWebHistory
// hash     =>  createWebHashHistory  
// abstact  =>  createMemoryHistory
const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出router
export default router
```

`main.vue`

```vue
<template>
  <el-card class="box-card ">
    <template #header>
      <div>
        <span>子路由</span>
        <router-link style="margin-left:10px;" to="/">childrenA</router-link>
        <router-link style="margin-left:10px;" to="/childrenB">childrenB</router-link>
      </div>
    </template>
    <router-view></router-view>
  </el-card>
</template>

<script setup lang="ts">

</script>

<style scoped>

</style>
```

## 命名视图

> - 类似于具名插槽。可对`<router-view name='xxx' />`指定不同的名称，默认视图名称是 `default`。
> - 命名视图可以在同一级（同一个组件）中展示更多的路由视图，而不是嵌套显示。 
> - 命名视图可以让一个组件中具有多个路由渲染出口，这对于一些特定的布局组件非常有用。 

`index.ts`

```typescript
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

const routes: Array<RouteRecordRaw> = [{
    path: '/root',
    component: () => import('@views/root.vue'),
    children: [
        {
            path: '',
            components: {
                default: () => import('@views/main.vue'),
                header: () => import('@views/header.vue'),
                bottom: () => import('@views/bottom.vue')
            }
        }
    ]
}]

const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出router
export default router
```

`root.vue`

```vue
<template>
  <router-view name="header"></router-view>
  <router-view></router-view>
  <router-view name="bottom"></router-view>
</template>

<script setup lang="ts">

</script>

<style scoped>

</style>
```

## 重定向

> `redirect`：重定向路由
>
> - 字符串：指定需要重定向地址
> - 对象：可根据需求配置 `path` 或 `name`
> - 函数：有一个路由参数，可返回字符串形式或对象形式
>
> ![image-20220525151327126](image/image-20220525151327126.png)

`index.ts` 

```
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

const routes: Array<RouteRecordRaw> = [{
    path: '/',
    name: 'Main',
    component: () => import('@views/main.vue'),
    // 字符串
    // redirect: '/childrenA',
    // 对象形式
    // redirect: {
    //   // path: '/childrenA'
    //   // name: 'childrenA'
    // },
    // 函数形式
    redirect: (to) => {
        console.log('to', to)
        
        // 返回字符串
        // return '/childrenA'
        // 返回对象
        return {
            path: '/childrenA',
            query: to.query
        }
    },
    children: [
        {
            path: '/childrenA',
            name: 'childrenA',
            component: () => import('@views/childrenA.vue')
        },
        {
            path: '/childrenB',
            name: 'childrenB',
            component: () => import('@views/childrenB.vue')
        }
    ]
}]

const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出router
export default router
```

## 路由别名

> `alias`：为路由指定别名，支持多个别名

`index.ts`

```typescript
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

const routes: Array<RouteRecordRaw> = [{
    path: '/',
    alias: ['/mian', '/mian1', '/main2'],
    component: () => import('@views/main.vue'),
    redirect: (to) => {
        return {
            path: '/childrenA',
            query: to.query
        }
    },
    children: [
        {
            path: '/childrenA',
            name: 'childrenA',
            component: () => import('@views/childrenA.vue')
        },
        {
            path: '/childrenB',
            name: 'childrenB',
            component: () => import('@views/childrenB.vue')
        }
    ]
}]

const router = createRouter({
    history: createWebHistory(),
    routes
})

//导出router
export default router
```

## 导航守卫

### 全局前置守卫

```typescript
/**
 * 路由前置守卫
 *  to：目标路由
 *  from：当前路由
 *  next()：放行路由跳转
 *  next(false)：阻止路由跳转
 *  next('/'):跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
 */
router.beforeEach((to, from, next) => {
    document.title = to.meta.title
    if(to.path === '/' || store.userStore.loginUser.username) {
        next()
    } else {
        next('/')
    }
})
```

### 全局后置守卫

后置守卫没有 `next()`

```typescript
router.afterEach((to, from) => {
    console.log('路由后置守卫');
    console.log(to);
    console.log(from);
})
```

## 路由元信息

通过路由记录的 `meta` 属性可以定义路由的**元信息**。使用路由元信息可以在路由中附加自定义的数据，例如：

- 权限校验标识。
- 路由组件的过渡名称。
- 路由组件持久化缓存 (keep-alive) 的相关配置。
- 标题名称

我们可以在**导航守卫**或者是**路由对象**中访问路由的元信息数据

```typescript
// TS 扩展
declare module 'vue-router' {
    interface RouteMeta {
        title: string
    }
}

const routes: Array<RouteRecordRaw> = [
    {
        path: '/',
        component: () => import('@views/Login.vue'),
        meta: {
            title: '登录'
        }
    },
    {
        path: '/userInfo',
        component: () => import('@views/UserInfo.vue'),
        meta: {
            title: '个人信息'
        }
    }
]

// 前置守卫根据元信息中 title 修改浏览器 title
router.beforeEach((to, from, next) => {
    document.title = to.meta.title
    if(to.path === '/' || store.userStore.loginUser.username) {
        next()
    } else {
        next('/')
    }
}
```

## 动态路由

> `const removeRoute = router.addRoute()`：
>
> - 可通过回调删除路由
> - 如果添加与现有途径名称相同的途径，会先删除路由，再添加路由
>
> `router.removeRoute()`：
>
> - 按名称删除路由
> - 当路由被删除时，**所有的别名和子路由也会被同时删除**
>
> `router.hasRoute()`：
>
> - 检查路由是否存在
>
> `router.getRoutes()`：
>
> - 获取一个包含所有路由记录的数组。
>
> **注意 vite 在使用动态路由的时候无法使用别名 @ 必须使用相对路径**

```typescript
const onSubmit = () => {
    formRef.value?.validate((valid, fields) => {
        if (valid) {
            store.userStore.LoginUser(form)
            initRouter()
            router.push('/userInfo')
        } else {
            console.log('error submit!', fields)
        }
    })
}

const initRouter = () => {
    const adminData = [
        {
            path: '/demo01',
            name: 'demo01',
            component: 'demo01.vue'
        },
        {
            path: '/demo02',
            name: 'demo02',
            component: 'demo02.vue'
        },
        {
            path: '/demo03',
            name: 'demo03',
            component: 'demo03.vue'
        }
    ]

    const userData = [
        {
            path: '/demo01',
            name: 'demo01',
            component: 'demo01.vue'
        }
    ]

    if(form.username === 'admin') {
        adminData.forEach(item => {
            router.addRoute({
                path: item.path,
                name: item.name,
                component: () => import(`../views/${item.component}`)
            })
        })  
    } else {
        userData.forEach(item => {
            router.addRoute({
                path: item.path,
                name: item.name,
                component: () => import(`../views/${item.component}`)
            })
        })  
    }

    console.log('---', router.getRoutes());

}
```

