---
title: Jest 测试框架搭建和简单使用
date: 2022-11-01 21:23:31
updated: 2022-11-01 21:23:31
tags:
  - jest
categories:
  - 前端工程化
  - 单元测试
---

### 了解测试用例和单元测试

- 测试用例：是为某个特殊目标而编制的一组测试输入、执行条件以及预期结果，测试是否满足特定需求；

- 单元测试：(是测试的级别)。unit testing 针对某一个功能的[最小部分(单元)]测试，比如(函数？类？)的执行结果是否符合预期。

  > 不同的企业可能对不同测试级别有不同的称谓，比如单元测试、增量测试、集成测试、回归测试、冒烟测试..... 谷歌对此创立了自己的命名方式：小型测试(具体到某个函数？)、中型测试(多个模块之间交互)、大型测试(端对端？系统整体验证)。

<!-- more -->

### 了解快照测试

> 快照测试：例如对 vue 的测试，就是将 vue 渲染的 dom 结果序列化成 string，然后存入到 snapshot 文件夹下，后缀为.snap, 如果是首次则会新建，不是首次，那么以后的每次测试，如果是基于快照的，就会那快照的内容与当前执行的内容做比较，如果不同，则抛出异常告知变更项；
> 快照测试一般用于代码趋于稳定的版本，提升测试的稳定性和速度；

### jest 环境搭建

```bash
mkdir jest-project
cd jest-project/

# 项目初始化及依赖安装
npm init -y
# jest 安装.. 注意版本号 28版本以后 需要另外再安装 jest-environment-jsdom
yarn add jest@24.8.0 --dev
# scripts 添加指令
test: "jest"
# test 文件改动 自动执行 jest
"test:debug": "jest --watchAll"

# 初始化 jest 配置文件 (npx xx 可以理解为寻址执行(优先找$path,再从node_modules查找)，如果xx存在，那么就执行这个exe， 不存在则安装再执行)
# 执行结束会根据选项,生成对应配置文件 jest.config.js|ts
npx jest --init
√ Would you like to use Typescript for the configuration file? ... no
√ Choose the test environment that will be used for testing » jsdom (browser-like)
√ Do you want Jest to add coverage reports? ... yes
√ Which provider should be used to instrument code for coverage? » babel
√ Automatically clear mock calls and instances between every test? ... yes

# 生成覆盖率检测文件
npx jest --coverage or 添加执行命令 yarn coverage: "jest --coverage"
```

#### 默认配置项

[`default-jest-config`](https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-plugin-unit-jest/presets/default/jest-preset.js)

### 测试用例简单示例

```javascript
// 添加测试的主文件 例如：feature1.js
// 测试对应测试文件 feature1.test.js

// 注意：jest 测试文件不需要具体指明文件名称 而是通过 test.js 标识

// feature1.js
function myFn1TestAdd(arg0, arg1) {
  return arg0 + arg1;
}

function myFn2TestLogic(arg0, arg1) {
  return arg0 < arg1;
}

module.exports = { myFn1TestAdd, myFn2TestLogic };

// feature1.test.js
const { myFn1TestAdd, myFn2TestLogic } = require('./feature1.js');

test('test number summation', () => {
  expect(myFn1TestAdd(99, 1)).toBe(100);
});

test('compare number', () => {
  expect(myFn2TestLogic(99, 1)).toBe(false);
  expect(myFn2TestLogic(99, 1)).not.toBe(true);
});
```

`理解：test(用例描述，执行回调) 创建一个测试用例， expect 预期(实际值：这里是测试的函数，那么即为函数的返回值) toBe[匹配器 通过 object.js(绝对比较，类似 === 效果) 比较](期望值)，not 可以理解为取反`

### 部分`Api`学习

- 自定义 matchers

​ `等值判断`

- `toEqual` 匹配器: 递归比较对象属性实例，和 `toBe`匹配器的区别在于 `toBe` 类似 `===` 的方式进行绝对匹配，实际运用中 `toEqual`更适合对比引用类型数据的预期输出；

- `toStrictEqual`匹配器：严格比较，例如：`[, , 1] 和 [undefined, undefined, 1]`

  ```javascript
  // expect({ a: 1, b: 'aa', c: false }).toEqual({ a: 1, b: 'aa' }); // false
  expect({ a: 1, b: 'aa', c: false }).toEqual({ a: 1, b: 'aa', c: false }); // true
  expect({ a: 1, b: 'aa', c: false }).toStrictEqual({ b: 'aa', a: 1, c: false }); // true

  expect({ a: 1, b: 'aa', c: false, __proto__: { q: 'eq' } }).toEqual({ b: 'aa', a: 1, c: false }); // true
  expect({ a: 1, b: 'aa', c: false, __proto__: { q: 'eq' } }).toStrictEqual({
    b: 'aa',
    a: 1,
    c: false,
  }); // true

  expect([, , 1]).toEqual([undefined, undefined, 1]); // true
  // expect([, , 1]).toStrictEqual([undefined, undefined, 1]); // false
  ```

- `toBeCloseTo(number, numDigits?)` 匹配器： 解决`JavaScript` 浮点数相等比较问题

  ```javascript
  test('test toBeCloseTo', () => {
    expect(0.1 + 0.2).toEqual(0.3); // false
    expect(0.1 + 0.2).toBeCloseTo(0.3); // true
  });
  ```

​ `数字大小比较`

- `toBeGreaterThan` `toBeGreaterThanOrEqual` `toBeLessThan` `toBeLessThanOrEqual` (参数均为`number | bigint`)匹配器: 是否大于、大于等于、小于、小于等于值；

  ```javascript
  test('test toBeLessThan toBeGreaterThan', () => {
    const a = 100;
    expect(a).toBeLessThan(101); // true

    expect(a).toBeGreaterThan(99); // true
  });
  ```

​ `类型判断`

- `toBeNull` `toBeUndefined` `toBeDefined ` 匹配器(无参数)：判断这个值是否为 `null、undefined、不为undefined`；

  ```javascript
  test('test undefined', () => {
    let a;
    expect(a).toBeUndefined(); // true

    a = undefined;
    expect(a).toBeUndefined(); // true

    a = void 0;
    expect(a).toBeUndefined(); // true
  });
  ```

- `toBeTruthy` `toBeFalsy` `toBeNaN` 匹配器(无参数)：判断一个值经过隐式转换后为 `true、false`(类似 1，'test' 等等 `toBeTruthy()` 则达到预期)、判断一个值是否为 `NaN`(和`toBeTruthy/toBeFalsy` 不同的是，这里为显示比较);

```javascript
test('test NaN', () => {
  let n = '11';
  expect(n).toBeNaN(); // false

  n = '11ds';
  expect(n).toBeNaN(); // false

  n = NaN;
  expect(n).toBeNaN(); // true
});
```

- `toBeInstanceOf(class)`匹配器：判断是否为某个类的实例对象；

​ `判断是否包含某个值`

- `toMatch(regexp | string) `匹配器：判断字符串是否能够根据`match`参数提取片段(简言之，`match` 为 `regexp` 时，类似`reg.test(str)`, 类型为 `string` 时类似 `str.includes(match) or str.indexOf(match) > -1`);

- `toMatchObject(obj)` 匹配器：判断 obj 是否为某个对象的子集

  ```typescript
  const obj = { be: 4, kit: { am: ['oven', 'stove', 'washer'], area: 20 } };
  const ob = {
    kit: {
      am: ['oven', 'stove', 'washer'],
    },
  };
  test('the  has my desired features', () => {
    expect(obj).toMatchObject(ob); // pase
  });
  ```

- `toContain(item)` 匹配器：判断数组是否包含了某个 `item`,类似 `arr.includes('xx')`;

  ```typescript
  test('test toContain', () => {
    expect(['aa', 'bb', 'cc']).toContain('aa'); // true
  });
  ```

​ `判断是否抛出异常`

- `toThrow()` 匹配器：判断`expect`传入值执行过程中发生异常; 注意 `throw 抛出的异常值 要同 toThrow 参数一致`。此时 `expect 传入的不再是具体的值，而是可执行回调`

  ```javascript
  test('test toThrow', () => {
    const fn1 = () => {
      throw new Error('aa');
    };

    const fn2 = () => {
      try {
        throw new Error('aa');
      } catch (error) {
        console.error(error);
      }
    };

    const fn3 = () => {
      throw new Error();
    };

    expect(fn1).toThrow('aa'); // true
    // expect(fn2).toThrow('aa'); // false
    expect(fn3).toThrow(); // true
    // expect(fn3).toThrow('aa'); // false
  });
  ```

### 让`jest` 支持 `esModule`

#### 方案一：添加 `babel` 转译

​ `jest 运行在 node 环境中，遵循 commonJs 规范，无法识别 EsModule 类似 export/import ... form ...模块导入导出，因此添加 babel 实现转译，将 EsModule 转换为 commonJs 的导入导出规范；`

- `@babel/core@7.4.5` 指定所有的转译都是用本地配置文件 `.babelrc or package.json`；

- `@babel/preset-env@7.4.5` 转换器核心，用作语法转换编译；

  > yarn add @babel/core@7.4.5 @babel/preset-env@7.4.5 --dev

- `yarn test` 执行 `jest` 前，`jest 的 babel-jest` 组件会优先检测是否安装了 `babel`，检测 `.babelrc` 配置文件是否存在，然后再根据 `babel`转译后的结果执行`test`

- 添加 `.babelrc` 配置文件，指定预设规则；`对象：presets 为['presetName', options: any][]类型，每个 item 为一个预设`，预设：就是提供一个预定的配置项，然后与当前配置做合并；

  ```json
  {
    "presets": [
      [
        "@babel/preset-env",
        {
          "targets": {
            "node": "current"
          }
        }
      ]
    ]
  }
  ```

#### 方案二：配置 `package.json`

- 将 `package.json` 的 `type` 字段设置为 `module`

- 每次执行 `jest` 指定环境变量

  > // package.json
  >
  > {
  >
  > // ...,
  >
  > "type": "module",
  >
  > "scripts": {
  >
  > ​ `"test": "NODE_OPTIONS=--experimental-vm-modules jest"`
  >
  > }
  >
  > }

### 测试 `typescript`

#### 方案一：仅编译，不做类型检测

`通过 @babel/preset-typescript 预设，完成 typescript 的转译`

> yarn add @babel/preset-typescript --dev
>
> // 配置 .babelrc 转换器规则
>
> { presets: [ ['@babel/preset-typescript'] ] }

#### 方案二：编译并对类型检测

`普通的 javascript 测试，使用内置的 babel-jest 组件。那么为 typescript 编写测试用例，则需要安装 ts-jest 组件；与此同时，安装 jest 的类型依赖，用于 .test.ts 中类型的注入。`

> // 建议 @types/jest 版本同 jest 主次版本一致
>
> yarn add ts-jest @types/jest --dev
>
> // 配置 `jest.config.js` , 添加 preset: `'ts-jest'` 预设配置：即为预配置项。由 `ts-jest` 提供配置项，然后与当前配置做合并，这也是为什么加了预设过后，当前配置文件就不需要再向 `moduleFileExtensions` 内添加 ts 后缀识别，也不需要添加` transform {'^.+\\.tsx?$': tsJest}`；告诉 `jest, .tsx 文件通过 require.resolve('ts-jest') 编译处理`
>
> `{ ...otherOptions, preset: 'ts-jest' }`

### 测试异步结果

`asyncFuntion.ts`

```typescript
export const myRequestFn1 = () => {
  return new Promise((res, rej) => {
    setTimeout(() => {
      res({ success: true });
    }, 1000);
  });
};
```

#### 方案一：接受一个参数 `done`

`不管 done 在 test 回调函数内是否有被使用，整个回调都会被判定为异步函数，需要等待 done() 结束执行。有点类似 promise.resolve 结束 promise。等待超时后结果为 false`

```typescript
// 直接在回调内的 promise.then 内写的语句不会被 test 检测，无关 equal 的值，结果都是 pass
test('test Done', () => {
  myRequestFn1().then((data) => {
    expect(data).toEqual({ success: true });
  });
});

// 等待超时，结果异常
test('test Done', (done) => {
  myRequestFn1().then((data) => {
    expect(data).toEqual({ success: true });
  });
});

// 正常执行
test('test Done', (done) => {
  myRequestFn1().then((data) => {
    expect(data).toEqual({ success: true }); // true
    // expect(data).toEqual({ success: false }); // false
    done();
  });
});
```

#### 方案二：`return` 返回一个 `promise` 结果

`返回一个 promise 结果，test 代码也可以被正常执行`

```typescript
test('test async return', () => {
  return myRequestFn1().then((data) => {
    expect(data).toEqual({ success: true }); // true
  });
});
```

`雷：当 return 返回的 promise 不包含错误时，catch 不会被执行，那么 .catch 内的 expec 始终都会被 pass；此时，通过添加 expect 断言来判断是否覆盖了 catch 测试语句`

```javascript
test('test promise.catch return', () => {
  expect.assertions(1); // 参数为 expect 的执行此时

  return myRequestFn1().catch((error) => {
    expect(error).toBeUndefined();
  });
});
```

#### 方案三：`async await` (推荐)

```typescript
test('test async await', async () => {
  const res = await myRequestFn1();

  // expect(res).toBeUndefined(); // failed
  expect(res).toEqual({ success: true }); // pass
});
```

### 生命周期钩子函数

- `beforeAll` `beforeEach` `afterEach` `afterAll` ：所有测试用例开始之前执行、每个测试用例开始之前执行、每个测试用例结束之后执行、所有测试用例结束之后执行。

- 作用域:

  > 钩子函数在父级分组可作用于子级，类似继承
  >
  > 钩子函数同级分组作用域互不干扰，各起作用
  >
  > 先执行外部的钩子函数，再执行内部的钩子函数

  ```typescript
  // 分组和钩子函数理解
  // 父级分组
  describe('test', () => {
    beforeAll(() => {}); // a
    beforeEach(() => {}); // b 这里父级的钩子函数会作用于子分组, 比如 beforeEach 在子分组的每个 test 执行之前会被触发

    // 子分组
    describe('test a', () => {
      beforeAll(() => {}); // c
      beforeEach(() => {}); // d

      test('test a.a', () => {}); // e
      test('test a.b', () => {}); // f
    });

    describe('test b', () => {
      beforeAll(() => {}); // g
      beforeEach(() => {}); // h

      test('test b.a', () => {}); // i
      test('test b.b', () => {}); // j
    });
  });

  // a -> c -> b -> d -> e -> b -> d -> f -> g -> b -> h -> i -> b -> h -> j
  ```

  `简言之，同级从上到下执行，只有执行到该子级后，该子级的周期钩子函数才会被触发，例如 beforeALL；然后当该及的 beforeALL 钩子执行后，开始从最外向内级依次执行 beforeEach ,再从上到下执行该级的 test 用例。注意：after 钩子与 before 相反，after 钩子是由内向外级依次执行。 类似 vue 的父子组件的生命周期执行顺序`；

### 测试用例的分组

​ `当测试用例足够多时，过量的测试用例 不方便管理及查看。此时使用 describe(name, fn) 对测试用例分组；注意：分组后还可以进行子分组，最外层可以理解为一个 describe`。

```typescript
// 原:
test('test a.a', () => {});
test('test a.b', () => {});
test('test b.a', () => {});
test('test b.b', () => {});

// 分组后：(将原有的 test 用例拷贝到 describe 回调内即可完成分组)
describe('test a', () => {
  test('test a.a', () => {});
  test('test a.b', () => {});
});
describe('test b', () => {
  test('test b.a', () => {});
  test('test b.b', () => {});
});
```

### 其他 `API`

- `test.only(name, fn)` 当存在 `test.only` 时，其他的测试用例会被 skipped 跳过，只执行当前的测试用例，通常用于 `debug`

### 测试 `Vue` 组件

`配合 @vue/test-utils 提供的 api，完成 vue 组件的 TDD(单元) 测试`

- 在项目中，能够被正确识别的测试文件，通常被指定在 `**/tests/unit/**/*.spec.[jt]s?(x) or **/__tests__/*.[jt]s?(x) `

- `it`(断言)，`test`测试，类似 it。

- 通过 `@vue/test-utils 提供的 mount or shadowMount 函数来挂载并渲染 vue`。

- 由于 `jest` 默认配置包含了对 `@` 符号路径的映射，所以可以直接使用。

  ```typescript
  // xx.spec.ts
  import { mount, shadowMount } from '@vue/test-utils';
  import XX from '@/xxx/Xx.vue'; // 由于配置了引用后缀，这里可以不写 .vue

  describe('测试一个 XX 组件功能', () => {
    it('测试 A 函数', () => {
      const wrapper = shadowMount(XX);
      // 断言 add TDD
      expect(wrapper.text()).toMatch('测试');
    });
  });
  ```

- `shadowMount 和 mount` 的区别：[参考](https://github.com/holylovelqq/vue-unit-test-with-jest/issues/4)

  `渲染区别：`

  ```
  mount：踏踏实实的渲染，会将被测试组件中使用到的子子孙孙组件完全渲染。最终结果内肯定不存在自定义组件名作为标签名，包括插件提供的V-btn之类的dom结构，全部不存在，彻底渲染到<div><span><p>之类

  shallowMount：浅渲染，与mount相反，既不会渲染子组件，更不用提孙子辈的组件，仅限测试组件本身。子组件会原原本本的显示子组件的存根，当使用stubs的时候，子组件存根是可以自定义显示内容的，如果未自定义，将会显示子组件名-stub的标签名，如<my-componet-stub></my-componet-stub>
  ```

  `场景`：

  ```
  mount：多数情况出现在snapshot中使用，而且也只是部分snapshot中必须使用mount。下面将说明何种情况：当被测试组件内包含子组件，且子组件的dom结构通过slot在被测试组件内定义的时候，必须使用mount来测试snapshot。

  hallowMount：所有必须使用mount的情况以外都使用shallowMount，使用mount将更大的消耗时间和性能，
  ----> 实际上所有测试都可以不使用mount，但是鉴于测试的复杂性，有可能会出现用shallowMount怎么也报错的情况，但是改成mount就能测试通过
  ```

  `基本准则：所有使用 shallowMount 的地方都可以用 mount 替换，但是但凡能使用 shallowMount 的地方坚决不用 mount；`
