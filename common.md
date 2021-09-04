
# TypeScript 常见类型
---

## 文字类型

文字类型跟`const`声明常量的时候十分相似, 但`const`是一次性赋值, 并且初始化的时候必须有值.  
而文字类型则是初始化的时候可以没值, 但后续只能赋值固定的文字, 并且在赋值之前不可使用

```ts
let str: 'hello' // 文字类型

console.log(str) // 报错, 不能在赋值之前使用

str = 'hello' // 正确, 因为 str 正好可以赋值 'hello'

str = 'world' // 报错, 不能把 'world' 分配给 'hello'
```

## 联合类型

```ts
let id: string | number = 1

let sex: '男' | '女' = '男'

let friend: string | string[] = 'jack'
```

## Type

```ts
// 使用 type 定义类型
type InfoType = { name: string }
type CurrType = { age: number }

// 合并多个类型
type AnotherType = InfoType & CurrType & {
  add: (x: number, y: number) => number
}

let obj: AnotherType = {
  name: 'jack',
  age: 99,
  add: (x, y) => x + y
}
```

const configInfo 



## Interface

- `只读属性`: 通过`readonly`关键词来声明, 后续属性的值不能修改, 效果类似`const`
- `可选属性`: 意味着该变量可能是`undefined`, 所以处理的时候还需要额外判断该变量是否存在
- `接口继承`: 可以使用`extends`继承多个接口, 并且多个接口之间要用`,`分隔
- `索引签名<待完善>`: `[key: string]: any`

```ts
// 当出现相同名称的接口时, 类型就会合并
interface InfoType { name: string }
interface InfoType { info: string }

// 定义一个接口类型
interface CurrType {
  age?: number // 可选属性
  readonly sex: string // 只读属性
}

// 继承多个接口
interface AnotherType extends InfoType, CurrType {
  add(x: number, y: number): number // 函数类型, 方式1
  reduce: (x: number, y: number) => number // 函数类型, 方式2
}

let obj: AnotherType = {
  name: 'jack',
  sex: '男',
  info: 'xxx',
  add: (x, y) => x + y,
  reduce: (x, y) => x - y
}
```

## 函数类型

定义函数类型有很多种方式, 单行时返回值类型使用`=>`, 多行时返回值类型使用`:`

```ts
// 方式1: 声明变量时定义函数类型
let fn: (name: string) => void

// 方式2: 类型别名单行简写
type FnType1 = (name: string) => void

// 方式3: 类型别名多行写法
type FnType2 = {
  (name: string): void
}

// 方式4: 使用接口进行定义
interface FnType3 {
  (name: string): void // 函数本身的类型
  str: string // 函数属性
  num?: number // 可选属性
  readonly desc: string // 只读属性
  add(x: number, y: number): number // 函数属性定义方法1
  reduce: (x: number, y: number) => number // 函数属性定义方法2
}

function test(fn: FnType3) {
  fn.str = 'hello'
  fn.desc = '' // 报错, 因为只读属性只能在初始化时赋值
  fn.add(1, 2)
  fn.reduce(3, 4)
  fn('world')
}
```

### 构造器类型<待完善>

```ts
interface DateType {
  new (time: string): Date
}

let myTime: DateType = class extends Date {}

console.log(new myTime('2021')) // "2021-01-01T00:00:00.000Z"
```

### 函数重载

```ts
const query = document.querySelector
interface Config { box: HTMLDivElement | null }


// 使用`es5`的`function`实现函数重载
function fn1(id: string): Config
function fn1(el: HTMLDivElement): Config
function fn1(config: Config): Config
function fn1(prop: any): Config {
  if (typeof prop === 'string') return { box: query(prop) }
  if (prop.nodeName === 'DIV') return { box: prop }
  if (typeof prop === 'object' && prop.box) return prop
  return { box: null }
}
fn1(999) // 报错, 可以看到提示


// 使用`interface`实现函数重载
interface FnType {
  (id: string): Config
  (el: HTMLDivElement): Config
  (config: Config): Config
}
let fn2: FnType = function (prop: any): Config {
  if (typeof prop === 'string') return { box: query(prop) }
  if (prop.nodeName === 'DIV') return { box: prop }
  if (typeof prop === 'object' && prop.box) return prop
  return { box: null }
}
fn2(999) // 报错, 可以看到提示
```

### this

> 在JavaScript里`this`属于关键词, 不能作为参数使用, 所以TypeScript规定: 可以通过`(this: Window) => void`这样的方式来定义`this`的类型, 并且要放在所有参数的前面

```ts
// 定义函数中this的类型
interface Obj {
  fn1: (this: Obj) => () => void
  fn2: () => (this: Window | void) => void
}

let obj: Obj = {
  fn1: function () {
    return () => {
      console.log(this)
    }
  },
  fn2: () => {
    return function () {
      console.log(this)
    }
  }
}

obj.fn1()() // obj
obj.fn2()() // 严格模式: undefined, 非严格模式: window
```
#### No Implicit This 禁止隐式的this

> 如果开启了`noImplicitThis`选项 (默认为false), [下面代码](https://www.typescriptlang.org/play?#code/FAGwpgLgBAZgdlAvFAFASiQPigb2FAqAYwHs4BnE8AOhBIHMUIALAS3LWAF9hRIoSAIwBWSXPkIB6AFTSJhaVECb8YBnEwODGgLO0W7QN4+gaPVAPPKA5OUCFNoEhzQBSu8goqZtyALigB1VnAAmJAO5QAPlAA3ElZ3TkIbVG1HKAgATwAHMBIYKHo6QQBDEAAVezDw6UlreABGJ3QscXDw0goqMFoGO3Z8gi4AGmK4ACZyjERsHChJSShAOzNALASYAFc4IghWMkB75UBTuU0o-UBTc0ARyMB-BMAzaMApFWtwgCdIadOEGbmFskj7J1cPbz9A4NCq6u-ayho6RhRVrVHjhDpdADMThu80WCAqeG+BHOEEu8P6gxOSN+9UagLyWLa1h4PGA8AqIygXjcni8wCEwmopQpo2pLzpDKZPXQLKpNO89JEXIhPIwlIZgsZ8Ah1CIWRAKDZtLQvKVAqAA)里会出现两处报错
- [x] **noImplicitThis** *(Enable error reporting when `this` is given the type `any`.)*  



```ts
let fn = () => {
  // 报错: The containing arrow function captures the global value of 'this'
  console.log(this)
}

let obj = {
  /**
   * 这里指定this类型似乎没有用
   * (this: Window | void) => void
   * (this: typeof globalThis) => void
   */
  fn1: () => {
    // 报错: The containing arrow function captures the global value of 'this'
    console.log(this)
  },
  fn2: () => { // 普通function可以指定this类型来规避警告
    return function (this: Window | void) {
      console.log(this)
    }
  },
  fn3: function () {
    return () => {
      console.log(this)
    }
  }
}

fn() // window
obj.fn1() // window
obj.fn2()() // window
obj.fn3()() // obj
obj.fn3.call(window)() // window
```