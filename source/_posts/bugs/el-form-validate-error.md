---
title: ElForm 偶现表单字段校验异常
date: 2023-09-08 2023-09-08 17:33:10
updated: 2023-09-08 2023-09-08 17:33:10
tags:
  - elForm
  - rule
  - validate
categories:
  - 踩坑记录
  - ElementUI
---

### 浅谈一下

`elementUI`存在很多已知或者未知的问题需要我们去发现并解决，但不可否认它确实在曾经取得过很好的认可。

在使用 `el-form-item` 组件时，正常情况下，我们经常需要自定义校验规则来规范用户输入，比如这样：

```html
<el-form-item prop="name" label="姓名" :rules="rules.name">
  <el-input v-model="form.name"></el-input>
</el-form-item>
<el-form-item prop="age" label="年龄" :rules="rules.age">
  <el-input-number v-model="form.age"></el-input-number>
</el-form-item>
...
```

```ts
public rules = {
  name: [{ required: true, message: '姓名不能为空', trigger: 'blur' }],
  age: [{ required: true, message: '年龄不能为空', trigger: 'change' }],
}
```

当然你也可以把 `rules` 绑定到 `el-form` 上，因为两者在校验过程中起的作用和我所描述的问题并不冲突。

😏😏😏 问题的产生往往是来自于部分特殊需求：

<!-- more -->

### 场景复现

#### 需求

- `el-form-item` 包裹的子组件 `（自定义的独立页面组件）`能够由自身提供校验函数，然后由 `el-form-item` 触发校验

- 能够根据表单权限（比如只读、可写、隐藏）等动态生成校验规则。`（权限可以来自于组件的 prop）`

#### 实现

基于性能优化方面考虑，将 `rules` 绑定到了每个 `el-form-item` 而非 `el-form`。

> _如果 `rules` 绑定到了 `el-form`，那么当其中某个组件的权限发生变更时，需要重新对整个 form 的 rules 计算并赋值，会影响到整个父级的 wrapper 组件 rerender，当组件数量达到一定程度时(刚好我所在场景下就会有很多...有的多达上千)，会明显感觉到性能差异_

为了满足上面的需求，`rules` 是绑定的计算属性，通过计算实时赋予 `el-form-item` 组件新的 `rules`，实现如下：

```html
<el-form-item :prop="prop" :rules="formItemRules">
  <CustomChildren ref="childrenRef"></CustomChildren>
</el-form-item>
```

```ts
// CustomChildren 是动态组件，根据组件 tag 渲染
@Ref() public readonly childrenRef!: CustomChildren;

@Prop() public readonly prop!: string;
@Prop() public readonly privilege!: 'VIEW' | 'HIDDEN' | 'EDIT';

public get formItemRules() {
  // 只有可编辑时需要 rule
  if (this.privilege === 'EDIT') {
    // getFormItemRules： rules 构建函数，具体实现由需求决定
    return this.getFormItemRules();
  }
  return [];
}

public getFormItemRules() {
  // 找到子组件实例 --- 具体的组件层级不做深究，由组件结构决定
  const controlRef = this.childrenRef?.$children?.[0];

  const ruleList: ValidationRule[] = [];

  // eg. nameRules，实际实现需要考虑通用性
  if (this.prop === 'name') {
    ruleList.push({ required: true, message: '请输入姓名', trigger: 'blur' });
  }
  // ... other prop

  // add children customRules
  ruleList.push(...(controlRef?.customRules ?? []));
}

// children customRules：
// export default class Upload extends Vue {
//   protected customRules: ValidationRule[] = [
//     {
//       trigger: 'submit',
//       validator: (r, v, callback) => {
//         if (this.uploading) {
//           callback(new Error('文件正在上传中...'));
//         }
//         return callback();
//       },
//     },
//   ];
// }
```

是不是看上去没有任何的问题？嘿，我开始也是这么想的...

### 问题描述

#### 不触发校验

给 `el-form-item` 绑定了 `rules` 后，变更了值，但是校验并没有触发，这是为什么呢？

- 首先我怀疑是 `rules` 构建过程中出错了，没有传入正确的 `rules`.

通过 `console.debug` 以及`vue-devtools` 进行查看，发现 `rules` 是正确的...那么会不会是出在 `el-form-item` 自身呢？

- 翻阅 `el-form-item` 源码：[https://github.com/ElemeFE/element/blob/dev/packages/form/src/form-item.vue](https://github.com/ElemeFE/element/blob/dev/packages/form/src/form-item.vue)

发现了这么一句： ~~移除了其他干扰代码 😁😁😁~~

```js
methods: {
  getRules() {
    let formRules = this.form.rules;
    const selfRules = this.rules;
    const requiredRule = this.required !== undefined ? { required: !!this.required } : [];

    const prop = getPropByPath(formRules, this.prop || '');
    formRules = formRules ? (prop.o[this.prop || ''] || prop.v) : [];

    return [].concat(selfRules || formRules || []).concat(requiredRule);
  },
  addValidateEvents() {
    const rules = this.getRules();

    if (rules.length || this.required !== undefined) {
      this.$on('el.form.blur', this.onFieldBlur);
      this.$on('el.form.change', this.onFieldChange);
    }
  },
},
mounted() {
  if (this.prop) {
    // ...
    this.addValidateEvents();
  }
},
```

从上面可以很清晰的看到，在 `el-form-item` 组件 `mounted` 的时候会添加 `el.form.blur` 和 `el.form.change` 事件监听，由表单组件内部 `dispatch`。`然而...🤔🤔🤔，它有个很致命的前提：rules.length`

继续分析：`getRules` 中，会优先从 `formRules 和 selfRules` 读取 rules`，然后根据 `required`决定是否生成`require rule`，最后返回新的 `rules`。

**这也是为什么我说 `rules` 绑定到 `el-form` 和 `el-form-item` 和这个 `bug` 不冲突**

综上不难发现，由于我前面添加了 `formItemRules` 在非 `EDIT` 的时候，**返回了空的 `rules`，`el-form-item` 未绑定 `change | blur` 的回调，导致就算 `rules` 新增 `change` 类的 `trigger`，也不会触发即时校验**。

为了解决这个问题，我就添加了一个空对象的 `rules`，即：

```ts
public get formItemRules() {
  ...
  return [{}];
}
```

**你以为这个问题就算解决了？其实没有！ 🤗🤗🤗**

#### `refs.childrenRef no-reactive`

由于需要从子组件本身获取自定义的 `customRules` 属性，所以这里采用了 `refs` 去找组件实例，从实例属性中获取，然而...`refs` 并非是响应式的....~~乌鱼子啊兄弟们 😗😗😗~~

怎么解决？🤨 它不是响应式的那我们就用其他的代替咯...比如标志位：

```ts
public itemRulesReady = false;

public get formItemRules() {
  if (this.itemRulesReady && this.privilege === 'EDIT') {
    return return this.getFormItemRules();
  }
  return [{}];
}

public mounted() {
  this.itemRulesReady = true;
}
```

通过标志位 `itemRulesReady` 来标识组件实例 `ref` 已完成加载，又由于 `itemRulesReady` 是响应式的，变更后计算属性就会重新计算，这个问题也就解决了。

话又说回来，前面为什么我会说“不触发校验”没有彻底解决，请往下看...

#### `is not a string/number/array`

虽然...添加了 `[{}]` 后，会触发即时校验了，但是当我 `submit` 表单的时候，却出现了 `is not a string/number/array` 的错误。`why???😫😫😫`

只有继续调试了，通过 `debug` 找到出现该校验提示的组件，发现它的 `rules` 是之前我设置的 `[{}]`，嘿~难不成是因为我传了空对象的原因？

点击提交，代码执行调用了 `validateForm`，而 `validateForm` 的作用就是触发 `elForm.validate`，最后抛出校验结果：

```ts
private validateForm() {
  const form = this.$refs.formRef as ElForm;

  return new Promise<[boolean, FieldErrorList, boolean, FieldErrorList]>((resolve) => {
    form.validate((isValid, errors, isNoWarning, warnings) => {
      resolve([isValid, errors as FieldErrorList, isNoWarning, warnings as FieldErrorList]);
    });
  });
}
```

去翻源码...找到 `el-form` 的 `validate` 方法，**我这里移除了一些干扰代码并写了注释：比如不会进入的 `if` 语句和其他变量**

```ts
validate(callback) {
  let count = 0;

  this.fields.forEach(field => {
    // 这里的 field 为 el-form-item 的 ref 组件实例，调用组件自身的 validate 方法
    field.validate('', (message, field) => {
      // 当所有组件的 validate 方法执行完成后触发外部回调
      if (typeof callback === 'function' && ++count === this.fields.length) {
        callback(...);
      }
    });
  });
}

// 进入 el-form-item 的 validate 方法。 目前就是 age 所在组件的校验异常了...
validate(trigger, callback = noop) {
  const rules = this.getFilteredRule(trigger); // 这个方法是对 rules 进行过滤，返回的其实还是 [{}]
  const descriptor = {};

  descriptor[this.prop] = rules; // descriptor => { age: [{}] }

  const validator = new AsyncValidator(descriptor);
  const model = {};

  model[this.prop] = this.fieldValue; // age 值是数字类型 => model: { age: 18 }

  validator.validate(model, { firstFields: true }, (errors, invalidFields) => {
    this.validateMessage = errors ? errors[0].message : '';
    // 这里就是触发外部回调，并把校验失败的字段传递出去
    callback(this.validateMessage, invalidFields);
  });
}
```

好像越来越接近真相了，有点兴奋...😯😯😯，既然 `message` 是 `asyncValidator` 抛出的，那就找到 `AsyncValidator` 的 `validate` 实现 ~~篇幅有点长~~：

```ts
// new 了一个 AsyncValidator 实例
constructor(descriptor: Rules) {
  this.define(descriptor);
}
define(rules: Rules) {
  this.rules = {};

  // 这一步做完，this.rules = { age: [{}] }
  Object.keys(rules).forEach(name => {
    const item: Rule = rules[name];
    this.rules[name] = Array.isArray(item) ? item : [item];
  });
}

// 下面看 validate 实现，不用太关注其他配置，所以我删了些许代码
validate(source_: Values, o: any = {}, oc: any = () => {}): Promise<Values> {
  let source: Values = source_; // { age: 18 }
  let options: ValidateOption = o; // { firstFields: true }

  if (options.messages) {
    // ... 进入 else，设置 options.messages
  } else {
    options.messages = this.messages(); // _messages === defaultMessages ===  如下
    // {
    //   types: {
    //     string: '%s is not a %s',
    //     method: '%s is not a %s (function)',
    //     array: '%s is not an %s',
    //     object: '%s is not an %s',
    //     ...
    //   },
    //  ...
    // }
  }

  const series: Record<string, RuleValuePackage[]> = {};
  const keys = options.keys || Object.keys(this.rules); // ['age']

  keys.forEach(z => {
    const arr = this.rules[z]; // [{}]
    let value = source[z]; // 18
    arr.forEach(r => {
      let rule: InternalRuleItem = r; // {}

      // rule.validator: https://github.com/yiminghe/async-validator/blob/HEAD/src/validator/string.ts
      rule.validator = this.getValidationMethod(rule);

      // getValidationMethod(rule: InternalRuleItem) {
      //   ...
      //   return validators[this.getType(rule)] || undefined; // asyncValidator 提供的 string 默认校验函数
      // }
      // getType(rule: InternalRuleItem) {
      //   ...
      //   由于 rule 是个 {}，不存在 type, getType(rule) 返回了默认值 string
      //   return rule.type || 'string';
      // }
      rule.field = z; // age
      rule.fullField = rule.fullField || z; // age
      rule.type = this.getType(rule); // string
      series[z] = series[z] || []; // series: { age: [] }
      series[z].push({
        rule,
        value,
        source,
        field: z,
      }); // series: { age: [{ rule: { field: 'age', type: 'string', fullField: 'age'， validator: ... }, value: 18, source: { age: 18 }, field: 'age' }] }
    });
  });

  return asyncMap(
    series,
    options,
    (data, doIt) => {
      const rule = data.rule; // 拿到处理后的 rule: { field: 'age', type: 'string', fullField: 'age'， validator: ... }
      let res: ValidateResult;

      if (rule.asyncValidator) {
        // ...
      } else if (rule.validator) {
        // 执行 validate
        res = rule.validator(rule, data.value, cb, data.source, options);
      }
    },
    results => {},
    source,
  );
}

// 进入默认生成的 rule.validator 即 https://github.com/yiminghe/async-validator/blob/HEAD/src/validator/string.ts
const string: ExecuteValidator = (rule, value, callback, source, options) => {
  const errors: string[] = [];
  const validate =
    rule.required || (!rule.required && source.hasOwnProperty(rule.field)); // validate：true
  if (validate) {
    // ...
    if (!isEmptyValue(value, 'string')) {
      rules.type(rule, value, source, errors, options);
      // ...
    }
  }
  callback(errors);
};

// 进入 rules.type
const type: ExecuteRule = (rule, value, source, errors, options) => {
  const custom = ['integer', 'float', 'array', 'regexp', 'object', 'method', 'email', 'number', 'date', 'url', 'hex'];
  const ruleType = rule.type;

  // 下面都是做值和 ruleType 类型比对，只不过单独把 string 区分开了，string 类型直接通过 typeof 进行比对判断
  if (custom.indexOf(ruleType) > -1) {
    // ...
  } else if (ruleType && typeof value !== rule.type) {
    errors.push(
      // format 是将 %s is not a %s 中的特殊字符进行替换，返回一个完整的 error message
      format(options.messages.types[ruleType], rule.fullField, rule.type), // age is not a string
    );
  }
};
```

`ok...我滴妈 终于找到源头了...`

因为空的 `rules: [{}]` 会在 `asyncValidator` 内生成 `[{ type: string, validator: ... }]` 的默认 `rule`，那么怎么样才能让 `rules` 同 `[{}]` 不参与校验，同时也不会导致其他异常出现呢？**总不能根据组件的类型，动态生成 type 再传进去吧？**

细看上述源码，不可避免的会生成 `validator` 校验方法，目前能做的貌似只有让这个 `[{}]` 能够不影响校验结果，能够正常使用，那么就得先分析 `getValidationMethod` 方法了，毕竟所有的 `validator` 都是基于这个方法获取的：

```ts
getValidationMethod(rule: InternalRuleItem) {
  if (typeof rule.validator === 'function') {
    return rule.validator;
  }
  const keys = Object.keys(rule);
  const messageIndex = keys.indexOf('message');
  if (messageIndex !== -1) {
    keys.splice(messageIndex, 1);
  }
  if (keys.length === 1 && keys[0] === 'required') {
    return validators.required;
  }
  return validators[this.getType(rule)] || undefined; // 之前 [{}] 的步骤就走到这里，也就是返回了基于 string 的默认 validator 函数
}
```

很明朗了，只要不走默认的 `return` 貌似都能绕过去...

### 解决办法：

附上之前的临时解决办法：

```ts
public get formItemRules() {
  ...
  return [{}];
}
```

终极方案：

- **方案一：将 `[{}]` 替换为 `[{ required: false }]`**. （当只提供 `required`，的时候，`asyncValidator` 会采用默认的 `required` 的校验方法，即：）

```ts
if (keys.length === 1 && keys[0] === 'required') {
  return validators.required;
}

// validators.required 如下： 传入 false，永远都不会影响校验
const required: ExecuteRule = (rule, value, source, errors, options, type) => {
  if (
    rule.required &&
    (!source.hasOwnProperty(rule.field) || isEmptyValue(value, type || rule.type))
  ) {
    errors.push(format(options.messages.required, rule.fullField));
  }
};
```

- **方案二：传入自定义的 `validator`，将 [{}] 替换为 [{ validator: () => {} }]**

### 结语

> 如果你的项目在开发过程中有使用到 `elForm`，同时又有动态生成修改 `rules` 的需求：**首先必须要为绑定的 `rules` 提供默认值，且默认值不能为空，否则会导致校验异常；其次为满足提供的默认 `rules` 不影响校验结果，不能直接传入 `[{}]`，而是采用上面两种解决方法避免影响.**
