---
title: vuex-typescript 状态管理器
date: 2021-05-16 13:48:06
tags: [vuex, typescript]
# type: "categories"
categories: [大前端, Vue]
---

`伪代码，随手写的，并不完整 旨在提供一个整体思路`

### main.ts 入口

```typescript
/* 引入 */
import store from "./store";

/* 注册 store, App: App.vue, router: router.ts */
new Vue({ router, store, render: (h) => h(App) }).$mount("#app");
```

### @/store/index.ts 入口

```typescript
import Vue from "vue";
import Vuex from "vuex";

/*
 * 继续 import 各个模块的 store.ts， 如 user/index.ts
 */

/* 注册 vuex */
Vue.use(vuex);

/* 定义全局 $store.state interface */
export interface RootState {
  /* ... 全局本身的 state */
  count: number;
  /* 模块的 state */
  user: UserState;
}

/* 全局 store */
export default new Vuex.Store({
  state: { count: 0 },
  actions: {},
  mutations: {},
  getters: {},
  modules: { user }, // store 的 各个模块
  plugins: [], // 使用的插件
});
```

### 类型常量 .type.ts

```typescript
/* 按模块定义常量 global.types.ts */
export const userMutationTypes = {
  SET_USER_INFO: "SET_USER_INFO",
};
```

### 局部模块 user/index.ts

```typescript
import { RootState } from '@/store';
import { Module, MutationTree, ActionTree, ActionContext, GetterTree } from 'vuex';
import { userMutationTypes } from './user.type.ts';

/* 定义模块自身的 state interface */
export interface UserState {
    me?: User;
}

/* 定义局部的 mutations、actions... */
const mutations = MutationTree<UserState> = {
    [userMutationTypes.SET_USER_INFO](state: UserState, payload: string) {
        state.me = payload;
    }
}
const actions: ActionTree<UserState, RootState> = {
    async [xxx]({ commit, state }: ActionContext<UserState, RootState>) {
        // 异步请求 await ...
        commit('mutation-type', 'data');
    }
},
const getters: GetterTree<UserState, RootState> = {
    /* state 为当前 module的 state */
    xxx(state) {
        return '';
    },
},

/* 导出局部模块 store */
const module = Module<UserState, RootState> = {
    state: {},
    actions,
    mutations,
    getters,
    // ...
}

export default module;
```

### 相关操作

- actions: 定义好 actions， 在代码中 通过 dispatch 触发。

```typescript
import { userActions } from "@/store/global.types";

/* vue-ts-class */
this.$store.dispatch(userActions.GET_LOGIN_INFO);
```

- mutations: 定义好 mutations, 在代码中 commit 触发。

```typescript
/* 可以通过dispatch-actions，在 actions 内部 commit(userMutationTypes.SET_OPEN_INIT, 'xxx'); */

/* 代码中直接触发 */
this.$store.commit(userMutationTypes.SET_OPEN_INIT, "xxx");
```

- state: 通过提交 mutations， 修改 state。代码中获取 如下：

```typescript
/* ts */
(this.$store.state.global as GlobalState).curInitiateProcess;
/* js */
this.$store.state.global.curInitiateProcess;
```

- getters: 同 computed 计算属性。（vuex 的数据 不适合在代码中直接使用， 通过相关计算，得到自己想要的结果，然后作为计算值使用）。

```typescript
/* 代码中的使用场景 */
this.isManager = this.$store.getters.isManager;
// or
// <!-- template -->
// v-if="$store.getters.isManager"
```

### 辅助映射函数

`四种辅助函数，都是将参数值作为属性或方法挂载到当前vue实例，通过映射关系对应到 this.$store.state, 达到同步调用访问的效果。`

- mapState
- 工具函数，传入的对象作为返回值混入 computed。 传入的对象：key 作为 computed 属性名， key 对应的函数作为 handler。

```typescript
import { mapState } from 'vuex';
// ...
computed: {
    ...mapState({
        XXX: (state) => (state as RootState).user.me,
        // ...
    })
}
```

- mapGetters 同 mapState

```typescript
computed: {
	...mapGetters(['filteredList']),
}
```

- mapActions - 官方案例

```typescript
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ]),
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}
```

- mapMutations 同 mapActions

```typescript
methods: {
    ...mapMutations({
        crease: 'increase'// 将 `this.crease()` 映射为 `this.$store.commit('increase')`
    })
}
```

### 其他一些场景

- 通常 state 只被允许在 mutations 内修改，任何非 mutations 引起的 state 值修改都应 抛出错误，严格模式下也是这样。
- 那么不可避免有一些场景：

- v-model 绑定 state:我们可以这样

```typescript
<input :value="message" @input="updateMessage">

// ...
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}
```

- 或者利用 computed 的 get set 方法：

```typescript
<input v-model="message">

// ...
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```
