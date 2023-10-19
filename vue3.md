# Vue3

[TOC]

2023/10/19

## 0x1. Pre

### a. 创建脚手架

1. 首先安装 create-vue (nodejs 16以上)

   ```sh
   npm create vue@latest
   ```

2. 安装依赖并且运行即可

   ```sh
   npm install
   npm run dev
   ```

### b. Vue3特性

1. 组件式语法，需要在 script 标签上添加 `setup` 关键字

   ```vue
   <template>
     <button @click="addCount">{{ count }}</button>
   </template>
   
   <script setup lang="ts">
   import { ref } from 'vue';
   
   const count = ref(0);
   
   const addCount = () => {
     count.value++;
   }
   </script>
   ```

   > setup 是 vue 组件生命周期中的一个阶段，在 `beforeCreate()` 之前执行。

## 0x2. 组合式API

### a. reactive 和 ref 函数

两者的区别：

* `reactive()`：用于接收**对象类型**的数据，并且返回一个响应式的对象

  ```vue
  <template>
    Reactive: <button @click="addCountReactive">{{ state.count }}</button>
  </template>
  
  <script setup lang="ts">
  import { reactive } from 'vue';
  const state = reactive({count : 0});
  
  const addCountReactive = () => {
    state.count++;
  }
  </script>
  ```

* `ref()`：用于接收**普通/对象类型**的数据，但是必须通过 `.value` 才能进行修改。

### b. computed 计算属性

根据变化的属性得到新的属性，使用 `computed()` 函数

```vue
<template>
  Ref: <button @click="addCount">{{ count }}</button> {{ afterCount }}
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';

const count = ref(0);

const addCount = () => {
  count.value++;
}

const afterCount = computed(()=>{
  return count.value * count.value;
})
</script>
```

> 尽量不要在 computed 中修改 dom，避免直接修改计算属性，如果修改尽量使用 `watch` 函数。

### c. watch函数

作用：侦听一个或者多个数据的变化，变化时执行回调函数（分为立即执行和深度侦听）

* 监听单个数据

  ```tsx
  const count = ref(0);
  
  watch(count, (newValue, oldValue) => {
    // ... callback
  })
  ```

* 监听多个数据，使用数组包起来，**只要有一个数据发生变化就会执行**

  ```tsx
  watch([name, age], ([newName, newAge], [oldName, oldAge])=> {
      // callback
  })
  ```

* 立即执行，在监听器创建的时候就立即触发回调

  ```tsx
  watch(count, (newValue, oldValue) => {
    // ... callback
  }, {immediate: true})
  ```

* 深度监听，针对于**对象类型数据**，不开启深度监听修改对象属性不会触发回调，但是**这里不会传递旧值**

  ```tsx
  watch(count, () => {
    // ... callback
  }, {deep: true})
  ```

* 精确监听，针对于**对象类型数据中的某个字段**，如果字段也是对象类型，也需要使用 `deep`，否则不需要。

  ```tsx
  const obj = ref({ name: '123', age: 10 })
  
  const chgObj = () => {
    obj.value.age.a++;
  }
  
  watch(
    // 这里的监听
    () => obj.value.age,
    (newVal, oldVal) => console.log(newVal, oldVal)
  );
  ```

### d. 生命周期函数

<img src="vue3.assets/lifecycle.16e4c08e-16977020185914.png" alt="组件生命周期图示" style="zoom: 50%;" />

使用案例：

```vue
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  console.log(`the component is now mounted.`)
})
</script>
```

### e. 父子组件通信

**父传子**：

 子组件使用 `defineProps` 来定义属性，也可以绑定响应式数据，使用 `:` 绑定即可。[高级用法：Props校验](https://cn.vuejs.org/guide/components/props.html#prop-validation)

```vue
<script setup lang="ts">
defineProps({
    msg: String
})
</script>

<template>
    <h1>{{ msg }}</h1>
</template>
```

父组件：

```vue
<script setup lang="ts">
import Son from './Son.vue';
</script>

<template>
    My Son:
    <Son msg="123"/>
</template>
```

插槽元素，可以在子组件中使用 `<slot/>` 标签来接收来自父组件中的元素。

---

**子传父**：通过父组件向子组件传递回调函数

子组件使用 `defineEmits` 来定义回调函数。

```vue
<script setup lang="ts">
defineProps({
    msg: String
})

// 定义回调
const emit = defineEmits(['cb']);

const send = ()=>{
    // 触发回调，第一个是回调函数名称，其余回调函数参数
    emit('cb', 'AKA Monster')
}

</script>

<template>
    <h1>{{ msg }}</h1>
    <button @click="send">123</button>
</template>
```

父组件：

```vue
<script setup lang="ts">
import Son from './Son.vue';

const getSonInfo = (info : string) => {
    console.log(info);
}

</script>

<template>
    My Son:
    <Son msg="123" @cb="getSonInfo"/>
</template>
```

### f. 模板引用ref

模板引用可以指向Dom对象以及Vue组件。

```vue
<script setup lang="ts">
import { VueElement, onMounted, ref } from 'vue';
import Son from './Son.vue';

const getSonInfo = (info: string) => {
    console.log(info);
}

// 1. 首先指向一个 null
const h1Ref = ref(null);
const sonRef = ref(null);

// 3. 只有在元素加载完毕onMounted之后才能使用
onMounted(()=>{
    console.log(h1Ref.value);
    console.log(sonRef.value);
})

</script>

<template>
    <!-- 2. 使用 ref 指向对应的元素，会自动赋值给 ref -->
    <h1 ref="h1Ref">h1</h1>
    <hr>
    My Son:
    <Son msg="123" @cb="getSonInfo" ref="sonRef" />
</template>
```

如果是 Vue 组件，想要父组件直接使用需要使用 `defineExposes` 进行暴露属性

```tsx
// 子组件暴露
const age = ref(0);

defineExpose({ age })
```

### g. provide 和 inject 组件

作用：顶层组件向任意底层组件传递数据和方法，实现跨组件通信。

使用方法：

* 顶层组件：使用 `provide` 函数提供数据

  ```tsx
  provide('key', obj)
  ```

* 底层组件：使用 `inject` 函数使用数据

  ```js
  const value = inject('key')
  ```

## 0x3. Pinia

[Pinia](https://pinia.vuejs.org/zh/) 是 vue 的最新状态管理库，vuex的替代品。

### a. 准备工作

1. 安装

   ```sh
   npm install pinia
   ```

2. 在 Vue 中添加组件

   ```js
   const pinia = createPinia()
   app.use(pinia)
   ```

3. 开始使用

### b. 基本用法

定义 store：

```tsx
import { defineStore } from "pinia";

export const useHeight = defineStore('height', {
    state: ()=> ({height : 0, width: 0}),
    getters: {
        square: (state) => state.height * state.width,
        sides: (state) => state.height + state.width,
    },
    actions: {
        hadd() {this.height++},
        wadd() {this.width++}
    }
})
```

使用：

```tsx
const height = useHeight()
```

## 0x4. Nuxt

### a. 创建 Nuxt 工程

1. 初始化工程

   ```sh
   npx nuxi@latest init <proj-name>
   ```

2. Nuxt中使用pinia

   ```sh
   npm i @pinia/nuxt
   ```

   其他比如：[Tailwind CSS with Nuxt](https://tailwindcss.com/docs/guides/nuxtjs)

3. 在 `app.vue` 中写入

   ```vue
   <template>
     <NuxtLayout>
       <NuxtPage />
     </NuxtLayout>
   </template>
   ```

4. 就可以创建 pages 目录并且开始开发了。

### b. 向 Nuxt 工程中添加 Tauri

1. 将以下文件以及文件夹都存放在项目的 src 目录下

   * public
   * server
   * app.vue

2. 在 nuxt.config.ts 中修改源目录为 `./src`：

   ```tsx
   export default defineNuxtConfig({
     devtools: { enabled: true },
     srcDir: "./src"
   })
   ```

3. 添加 Tauri 到当前项目中

   首先需要安装 tauri

   ```sh
   # npm 安装 tauri
   npm install --save-dev @tauri-apps/cli
   # cargo 安装 tauri
   cargo install tauri-cli
   ```

   添加 tauri

   ```sh
   cargo tauri init
   ```

4. 配置 tauri.conf.json

   ```json
   {
     "build": {
       "beforeBuildCommand": "npm run generate",
       "beforeDevCommand": "npm run dev",
       "devPath": "http://localhost:3000",
       "distDir": "../dist"
     },
       // ....
   }
   ```

5. 在 package.json 中的 scripts 下添加：

   ```json
   "scripts": {
       // ...
       "tauri": "tauri"
       // ...
   },
   ```

6. 配置 nuxt.config.js，关闭 ssr

   ```json
   export default defineNuxtConfig({
     devtools: { enabled: true },
     srcDir: "./src",
     ssr: false,
     vite: {
       // Better support for Tauri CLI output
       clearScreen: false,
       // Enable environment variables
       // Additional environment variables can be found at
       // https://tauri.app/2/reference/environment-variables/
       envPrefix: ["VITE_", "TAURI_"],
     },
   });
   ```

7. 运行项目：

   ```sh
   cargo tauri dev
   ```