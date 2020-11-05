# 崔锐 | Part 3 | 模块五

## 1、Vue 3.0 性能提升主要是通过哪几方面体现的？

响应式系统升级

- Vue.js 2.x 中响应式系统的核心 definProperty （初始化过程）
- Vue.js 3.0 中使用 Proxy 对象重写响应式系统
    - 可以监听动态新增的属性
    - 可以监听删除的属性
    - 可以监听数组的索引和length属性

编译优化

- Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程
- Vue.js 3.0 中标记和提升所有 的静态根节点，diff 的时候只需要对比动态节点内容
    - Fragments（升级 vetur 插件）
    - 静态提升
    - Patch flag
    - 缓存时间处理函数

源码体积优化

- Vue.js 3.0 中移除了一些 不常用的  API
    - 例如： inline-templatet、filter 等
- Tree-shaking

## 2、Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？

Vue 2.x 在一些中小型项目很好用 ，但在开发长期迭代或大型项目中，有些限制。在大型项目中，会有一些复杂组件，有时候会很那看懂，原因是Vue 2.x采用的是 Options API。Options API 指的是使用一个包含描述组件的对象来创建组件的方式。

Options API

- 包含一个描述组件选项（data、methods、props 等）的对象
- Options API 开发复杂组件，同一个功能 逻辑的代码被拆分到不同选项

如果添加一个搜索功能，就得加入到各个方法中 ，就得不停的上下拖动 滚动条。Options API 还难以提取组件中可重用的逻辑。

Composition API

- Vue.js 3.0  新增 的 一组 API
- 一组基于函数的 API
- 可以更灵活的组织组件的逻辑

## 3、Proxy 相对于 Object.defineProperty 有哪些优点？

首先，defineProperty 的缺点主要有：

- Object.defineProperty无法监控到数组下标的变化，导致直接通过数组的下标给数组设置值，不能实时响应。
- Object.defineProperty只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历。Vue 2.x里，是通过 递归 + 遍历 data 对象来实现对数据的监控的。

Proxy是es6提供的新特性。优点有：

- 可以劫持整个对象，并返回一个新对象。
- 有13种劫持操作。
- Vue 3 底层采用 proxy 对象实现，初始化时不需要遍历所有属性，再通过给属性 defineProperty 转换成 getter，setter，提高了速度。

```jsx
const p = new Proxy(target, handler);

const handler = {
	get (target, key, receiver) { ... },
	set (target, key, value, receiver) { ... },
	deleteProperty (target, key) { ... }
}
```

## 4、Vue 3.0 在编译方面有哪些优化？

- Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程
- Vue.js 3.0 中标记和提升所有 的静态根节点，diff 的时候只需要对比动态节点内容
    - Fragments（升级 vetur 插件）
    - 静态提升
    - Patch flag
    - 缓存时间处理函数
- Vite 在开发模式下不需要打包可以直接运行（Vite 在生产环境下使用 Rollup 打包），更快。

## 5、Vue.js 3.0 响应式系统的实现原理？

**Vue.js 3 响应式优势**

- Vue 3 重写了响应式系统，与Vue 2 相比，Vue 3 底层采用 proxy 对象实现。
- 多层属性嵌套，在访问属性过程中处理下一级属性。
- Vue 3的响应式系统默认可以监听动态添加的属性
- 默认监听属性的删除操作
- 默认监听数组的索引 和 length 属性的修改操作。
- 可以作为单独的模块使用

模拟实现 Vue 3 的一些核心方法

- reactive / ref / toRefs / computed
- effect （watch 函数 runcore 中实现）
- track
- trigger

**Proxy 对象回顾**

```jsx
		// Reflect.getPrototypeOf()
		// Object.getPrototypeOf()
    const proxy = new Proxy(target, {
      get (target, key, receiver) { 
        // return target[key]
        return Reflect.get(target, key, receiver)
      },
      set (target, key, value, receiver) {
        // target[key] = value
        return Reflect.set(target, key, value, receiver)
      },
      deleteProperty (target, key) {
        // delete target[key]
        return Reflect.deleteProperty(target, key)
      }
    })
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    'use strict'
    // 问题1： set 和 deleteProperty 中需要返回布尔类型的值
    //        在严格模式下，如果返回 false 的话会出现 Type Error 的异常
	  const target = {
      foo: 'xxx',
      bar: 'yyy'
    }
    // Reflect.getPrototypeOf()
    // Object.getPrototypeOf()
    const proxy = new Proxy(target, {
      get (target, key, receiver) {      // receiver：当前的proxy对象或者继承的proxy对象
        // return target[key]
        return Reflect.get(target, key, receiver)
      },
      set (target, key, value, receiver) {
        // target[key] = value
        return Reflect.set(target, key, value, receiver)
      },
      deleteProperty (target, key) {
        // delete target[key]
        return Reflect.deleteProperty(target, key)
      }
    })

		// 如果 set 和 deleteProperty 没有return值，会返回 undefined，布尔值是 false，浏览器会报错。
    proxy.foo = 'zzz'
    // delete proxy.foo

    // 问题2：Proxy 和 Reflect 中使用的 receiver

    // Proxy 中 receiver：Proxy 或者继承 Proxy 的对象
    // Reflect 中 receiver：如果 target 对象中设置了 getter，getter 中的 this 指向 receiver

    const obj = {
      get foo() {
        console.log(this)
        return this.bar
      }
    }

    const proxy = new Proxy(obj, {
      get (target, key, receiver) {
        if (key === 'bar') { 
          return 'value - bar' 
        }
        return Reflect.get(target, key, receiver)
      }
    })
    console.log(proxy.foo)
  </script>
</body>
</html>
```

**Reactive**

- 接收一个参数，判断这参数是否是对象。（如果不是对象，直接返回；只能把对象转换成响应式对象，与ref不同）
- 创建拦截器对象 handler，设置 get / set / deleteProperty
- 返回 Proxy 对象

```jsx
const isObject = val => val !== null && typeof val === 'object'
const convert = target => isObject(target) ? reactive(target) : target
const hasOwnProperty = Object.prototype.hasOwnProperty
const hasOwn = (target, key) => hasOwnProperty.call(target, key)

export function reactive (target) {
  if (!isObject(target)) return target

  const handler = {
    get (target, key, receiver) {
      // 收集依赖
      track(target, key)
      const result = Reflect.get(target, key, receiver)
      return convert(result)
    },
    set (target, key, value, receiver) {
      const oldValue = Reflect.get(target, key, receiver)
      let result = true
      if (oldValue !== value) {
        result = Reflect.set(target, key, value, receiver)
        // 触发更新
        // trigger(target, key)
      }
      return result
    },
    deleteProperty (target, key) {
      const hadKey = hasOwn(target, key)
      const result = Reflect.deleteProperty(target, key)
      if (hadKey && result) {
        // 触发更新
        // trigger(target, key)
      }
      return result
    }
  }

  return new Proxy(target, handler)
}

```

**Effect & Track**

```jsx
let activeEffect = null
export function effect (callback) {
  activeEffect = callback
  callback() // 访问响应式对象属性，去收集依赖
  activeEffect = null
}

let targetMap = new WeakMap()

export function track (target, key) {
  if (!activeEffect) return
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  dep.add(activeEffect)
}
```

**Trigger**

```jsx
export function trigger (target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return
  const dep = depsMap.get(key)
  if (dep) {
    dep.forEach(effect => {
      effect()
    })
  }
}
```

**ref**

ref 函数参数接收原始值或者对象。如果传入的是对象，并且这个对象是ref创建的，直接返回；如果是普通对象，内部会调用reactive创建响应式对象，否则只创建一个value属性的响应对象。

```jsx
export function ref (raw) {
  // 判断 raw 是否是ref 创建的对象，如果是的话直接返回
  if (isObject(raw) && raw.__v_isRef) {
    return
  }
  let value = convert(raw)
  const r = {
    __v_isRef: true,
    get value () {
      track(r, 'value')
      return value
    },
    set value (newValue) {
      if (newValue !== value) {
        raw = newValue
        value = convert(raw)
        trigger(r, 'value')
      }
    }
  }
  return r
}
```

**toRefs**

toRefs 函数接收reactive返回的响应式对象，也就是一个 proxy 对象。如果传入的参数不是reactive响应式对象，直接返回。然后再把传入的所有属性，转换为类似于ref返回的对象，把转换后的属性挂载到一个新的对象上返回。

```jsx
export function toRefs (proxy) {
  const ret = proxy instanceof Array ? new Array(proxy.length) : {}

  for (const key in proxy) {
    ret[key] = toProxyRef(proxy, key)
  }

  return ret
}

function toProxyRef (proxy, key) {
  const r = {
    __v_isRef: true,
    get value () {
      return proxy[key]
    },
    set value (newValue) {
      proxy[key] = newValue
    }
  }
  return r
}
```

**computed**

computed 需要接受一个有返回值的函数作为参数，这个函数的返回值就是计算属性的值，并且需要监听这个函数内部使用的响应式变化，最后，把这个函数执行的结果返回。

```jsx
export function computed (getter) {
  const result = ref()

  effect(() => (result.value = getter()))

  return result
}
```