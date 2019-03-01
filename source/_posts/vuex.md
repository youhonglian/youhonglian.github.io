---
title: 对Vuex的基本理解
categories:  vue.js
tags: 
- vuex
---

## Vuex
> vuex是为vue.js开发的状态管理模式，存储应用中的所有数据状态，便于统一管理。它是应用中唯一的数据源，所以一个应用中只有一个store实例。


### 安装和使用

```JavaScript
import Vue from 'vue'

import Vuex from 'vuex'

//Vuex用过store选项，提供了一种机制，将数据从跟组件注入到每个子组件中（需要调用Vue.use(Vuex)）:

Vue.use(Vuex)
```



### state

通过在根实例中注册store选项，该store实例会注入到根组件下的所有子组件中。且子组件能通过this.$store访问到

```JavaScript 
 const Counter = {
      template: `<div>{{ count }}</div>`,
      //从 store 实例中读取状态最简单的方法就是在计算属性中返回某个状态：
      computed: {
          count () {
          return this.$store.state.count
          }
      }
  }
 
```



### mapState辅助函数

当一个组件需要获取多个状态时，逐个声明计算属性会很麻烦，为此我们可以使用mapState辅助函数帮我们生成： // 在单独构建的版本中辅助函数为 Vuex.mapState

```JavaScript
import { mapState } from 'vuex'

export default {
  computed: mapState({
         /*三种不同的方式*/
    // 箭头函数可使代码更简练
    count: state => state.count,
    // 传字符串参数 'count' 等同于 `state => state.count`
        countAlias: 'count',
        // 为了能够使用 `this` 获取局部状态，必须使用常规函数
        countPlusLocalState (state) {
          return state.count + this.localCount
        }
  })
}
```

上边我们给mapState传了一个对象，如果我们要生成的计算属性名称与state子节点名称相同时，也可以直接给mapState传入一个字符串数组：

```JavaScript
   computed: mapState([
      // 映射 this.count 为 store.state.count
      'count'
    ])
```

### 对象展开运算符

mapState函数返回的是一个对象，但是一个组件中的计算属性，不仅有来自store的，还有它局部的。那么如何混用呢？我们使用对象展开运算符：

```JavaScript 
 computed: {
      //localComputed 是组件的局部计算属性
      localComputed () { /* ... */ },
      // 使用对象展开运算符将此对象混入到外部对象中
      ...mapState({
        // ...
      })
    }
    
```


### Getters

有时候我们需要从store中的state中派生出一些状态，例如对列表进行过滤并计数：

```JavaScript
     computed: {
      doneTodosCount () {
        return this.$store.state.todos.filter(todo => todo.done).length
      }
   }
```
Vuex允许我们在store中定义getters（可以认为是store的计算属性）。getter接受state作为第一个参数：

```JavaScript
// 在'store/index.js'中
    const store = new Vuex.Store({
      state: {
        todos: [
          { id: 1, text: '...', done: true },
           { id: 2, text: '...', done: false }
            ]
      },
      getters: {
            doneTodos: state => {
          return state.todos.filter(todo => todo.done)
        },
    // getters也可以接受其他getters作为第二个参数
        doneTodosCount: (state, getters) => {
          return getters.doneTodos.length
            }
      }
    })
```

### mapGetters辅助函数

mapGetters辅助函数仅仅是将store中的getters映射到局部计算属性：
```JavaScript
    import { mapGetters } from 'vuex'

    export default {
    // ...
    computed: {
    // 使用对象展开运算符将 getters 混入 computed 对象中
    ...mapGetters([
    'doneTodosCount',
    'anotherGetter',
    // ...
    ])
    //如果想给getter属性领取一个名字，可以对象形式：
    mapGetters({
    // 映射 this.doneCount 为 store.getters.doneTodosCount
    doneCount: 'doneTodosCount'
    })
    }
    }
```

### Mutations

更改Vuex的store中的状态的唯一方法是提交mutation。Vuex的mutataions非常类似于事件：每个mutation都有一个字符串的事件类型（type）和一个回调函数（handler）。这个回调函数就是我们实际进行状态更改的地方，他会接受state作为第一个参数：

```JavaScript
    const store = new Vuex.Store({
    state: {
            count: 1
    },
    mutations: {
            increment (state) {
        // 变更状态
        state.count++
            }
    }
    })
```

上边注册了一个类型为increment的mutation：“当触发一个类型为increment的mutation时，调用此函数。”实际使用时：store.commit('increment')

### 使用常量替代Mutation事件类型

使用常量替代 mutation 事件类型在各种 Flux 实现中是很常见的模式。这样可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个 app 包含的 mutation 一目了然：

```JavaScript 
  //mutation-types.js

  export const SOME_MUTATION = 'SOME_MUTATION'
```

```JavaScript
    //store.js
    import Vuex from 'vuex'
    import * as types from './mutation-types'

    const store = new Vuex.Store({
    state: { ... },
    mutations: {
      // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
      [types.SOME_MUTATION] (state) {
      // mutate state
      }
     }
    })
```

mutation必须是同步函数你可以在组件中使用this.store.commit('type')提交mutataion，或者使用mapMutations辅助函数将组件中的methods映射为store.commit调用（需要在根节点注入store）。

```JavaScript
    import { mapMutations } from 'vuex'

    export default {
    // ...
    methods: {
      ...mapMutations([
      'increment' // 映射 this.increment() 为 this.$store.commit('increment')
    ]),
    ...mapMutations({
       add: 'increment' // 映射 this.add() 为 this.$store.commit('increment')
    })
    }
    }
```

### Actions

Action类似于mutation，不同之处在于：

Action提交的是mutataion，而不是直接变更状态。
Action可以包含任意异步操作。
注册一个简单的Action：

```JavaScript
    const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        increment (state) {
        state.count++
        }
    },
    actions: {
        increment (context) {
        context.commit('increment')
        }
    }
    })
```

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。当我们在之后介绍到 Modules 时，你就知道 context 对象为什么不是 store 实例本身了。

>解构：ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值。

```JavaScript
    //例如：赋值  var [a,b,c] = [1,2,3]
    //例如：交换变量 [x,y] = [y,x]
    //例如：函数参数解构：
    // function add([x, y]){
    //   return x + y;
    // }
    //实践中我们常用到 参数解构 来简化代码，下边
    //的 { commit } 就用到了解构。

    actions: {
    increment ({ commit }) {
        commit('increment')
    }
}
```
这里解构的对象是context对象，也就是说context <==> { commit }，这样写就可以用commit替代context.commit，简化代码。

### 分发Action

Action通过store.dispatch方法触发：store.dispatch('increment') Actions 支持同样的载荷方式和对象方式进行分发：

* 以载荷形式分发
```JavaScript
    store.dispatch('incrementAsync', {
        amount: 10
    })
```
* 以对象形式分发
```JavaScript
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```
我们可以在Action内部执行异步操作：
```JavaScript
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```
* 在组件中分发Action

你在组件中使用 this.$store.dispatch('xxx') 分发 action，或者使用 mapActions 辅助函数将组件的 methods 映射为 store.dispatch 调用（需要先在根节点注入 store）：
```JavaScript
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
       'increment' // 映射 this.increment() 为 this.$store.dispatch('increment')
    ]),
    ...mapActions({
       add: 'increment' // 映射 this.add() 为 this.$store.dispatch('increment')
    })
  }
}
```

### Modules
使用单一状态树，导致应用的所有状态集中到一个很大的对象。但是，当应用变得很大时，store 对象会变得臃肿不堪。

为了解决以上问题，Vuex 允许我们将 store 分割到模块（module）。每个模块拥有自己的 state、mutation、action、getters、甚至是嵌套子模块——从上至下进行类似的分割：

```JavaScript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```
