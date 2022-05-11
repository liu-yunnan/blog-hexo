---
title: vuex
date: 2021-05-10
tags: [vue]
cover: https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204131011473.png
---
# vuex

实现组件全局状态（数据）管理的一种机制，方便实现组件之间的数据共享

- 父向子传值：v-bind
- 子向父传值：v-on
- 兄弟组件之间共享数据：EventBus
  - $on接收数据的那个组件
  - $emit发送数据的那组件

好处：

1. 集中管理共享数据，易于开发和后期维护
2. 高效实现组件之间的数据共享，提高开发效率
3. vuex中的数据都是**响应式**的，能够实时保持数据与页面的**同步**

需要共享的数据，有必要存在vuex，一般情况，组件的私有数据，放入data中

## 核心概念

### State:提供唯一的**公共数据源**

所有共享的数据都要统一放到Store的State中进行存储

1. 第一种方法：this.$store.state.数值名称

```vue
export default new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})


<template>
  <div>
    count:{{$store.state.count}}
    <button @click="sub">-1</button>
  </div>
</template>

<script>
export default {
  methods: {
    sub () {
      this.$store.state.count -= 1
    }
  }
}
</script>
```

2.第二种方法：

- 从vuex中按需导入mapState函数。
- 通过导入的mapState函数，将当前组件需要的全局数据映射为当前组件的computed计算属性

```vue
------------------------------vuex----------------------------------
export default new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})

------------------------------组件----------------------------------
<template>
  <div>
    count:{{count}}
  </div>
</template>

<script>
import { mapState } from 'vuex'
export default {
  computed: {
    // 将全局数据映射为计算属性
    ...mapState(['count'])
    // ...展开运算符
  }
}
</script>

```

### Mutation:用于**变更Store中的数据**

1. **只能通过mutation变更Store数据，不可以直接操作Store中的数据**
2. 通过这种方式虽然操作起来繁琐，但是可以**集中监控所有数据变化**

1.触发的第一种方法

 this.$store.commit()

```vue
------------------------------vuex----------------------------------

export default new Vuex.Store({
  // 唯一的公共数据源，所有共享的数据都要统一放到Store的State中进行存储
  state: {
    count: 1
  },
  mutations: {
    // 传参
    addN (state, step) {
      state.count += step
    },
    // 不传参
    sub (state) {
      state.count -= 1
    }
  },
})

------------------------------组件1----------------------------------
<script>
export default {
  methods: {
    add () {
      // commit 的作用，就是调用 某个mutation函数
      this.$store.commit('addN', 3)
    }
  }
}
</script>
------------------------------组件2----------------------------------
 methods: {
    sub () {
      // 触发mutations的第一种方式
      this.$store.commit('sub')
    }
  }
```

2.触发的第二种方式

```vue
<script>
import { mapMutations } from 'vuex'

export default {
  methods: {
    // sub () {
    //   // 触发mutations的第一种方式
    //   this.$store.commit('sub')
    // }
    ...mapMutations(['sub', 'subN']),
    handle1 () {
      // this.sub()
        //sub也可在button上直接绑定
      this.subN(3)
    }
  }
}
</script>
```

延时操作

##### motations函数中不能执行异步操作

### Action:用于**异步处理任务**

- 异步操作变更数据：必须通过action，不能使用motations，但是在action中还是要通过触发mutation的方式间接变更数据

```vue
// 在action中要通过触发mutation的方式间接变更数据
  actions: {
    addasync (context) {
      setTimeout(() => {
        context.commit('add')
      }, 1000)
    }
  },
```

触发actions异步任务时 **携带参数**

```vue
 addasync1 (context, step) {
      setTimeout(() => {
        context.commit('addN', step)
      }, 1000)
    }
```

1. 触发actions的第一种方式

   this.$store.dispatch()

2. 触发actions的第二种方式

   按需导入mapActions函数，将需要的actions函数，映射为当前组件的methods方法

   ```vue
      <button @click='addasync'>async+1</button>
   <button @click='addasync1(5)'>async+n</button>
   import { mapActions } from 'vuex'
   methods: {
       ...mapActions(['addasync', 'addasync1']),
     }
   ```

   

### Getter:对Store中的数据进行加工处理形成新的数据

1. 对Store中已有的数据加工处理之后形成新的数据，类似**vue的计算属性**

2. Store中数据发生变化，Getter的数据也会跟着变化

3. 使用getters的第一种方式：

   this.$store.getters.名称

4. 第二种方式：

   ```vue
   import { mapGetters } from 'vuex'
   computed: {
       ...mapGetters(['showNum']),
     }
   ```