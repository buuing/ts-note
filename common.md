
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

### 索引签名

##### **索引签名和属性同时使用**

```ts
// 因为 sex 也归属于字符串索引, 所以它必须是字符串索引的子类型, 或与其保持一致类型
interface Obj {
  [key: string]: string | number
  name: string
  age: number
  sex: boolean // 报错: 不能将 boolean 类型分配给 string | number 类型
}
```

##### **两种索引签名同时使用**

索引签名的类型必须是字符串或数字, **并且当同时使用两种签名时, 数字索引必须是字符串索引的子类型**, 这是因为js内部会自动将数字转化成字符串
也就是说`obj[99]`和`obj['99']`是等价的

```ts
// 父类型
interface Parent { name: string }
// 子类型
interface Child extends Parent { age: number }

// 同时使用两种索引签名时, 数字索引的返回类型必须是字符串索引返回类型的子类型
interface Obj {
  [key: string]: Parent
  [key: number]: Child
}

let obj: Obj = {}
obj['hello'] = { name: 'jack', age: 10 } // 报错: Parent 上没有 age 属性
obj['999'] = { name: 'jack' } // 报错: Child 类型上缺少 age 属性
obj[1000] = { name: 'jack' } // 报错: Child 类型上缺少 age 属性
```

## 函数类型

定义函数类型有很多种方式, 单行时返回值类型使用`=>`, 多行时返回值类型使用`:`

```ts
// 方式1: Function 全局类型
// 不推荐, 因为通过 Function 函数返回值的类型是一个 any
let fn1: Function

// 方式2: 声明变量时定义函数类型
let fn2: (name: string) => void

// 方式3: 类型别名单行简写
type FnType1 = (name: string) => void

// 方式4: 类型别名多行写法
type FnType2 = {
  (name: string): void
}

// 方式5: 使用接口进行定义
interface FnType3 {
  (name: string): void // 函数本身的类型
  str: string // 函数属性
  num?: number // 函数可选属性
  readonly desc: string // 函数只读属性
  add(x: number, y: number): number // 函数的方法1
  reduce: (x: number, y: number) => number // 函数的方法2
}

function test(fn: FnType3): void {
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

> 如果开启了[**`noImplicitThis`**](https://www.typescriptlang.org/tsconfig#noImplicitThis)选项 (默认为false), [下面代码](https://www.typescriptlang.org/play?#code/FAGwpgLgBAZgdlAvFAFASiQPigb2FAqAYwHs4BnE8AOhBIHMUIALAS3LWAF9hRIoSAIwBWSXPkIB6AFTSJhaVECb8YBnEwODGgLO0W7QN4+gaPVAPPKA5OUCFNoEhzQBSu8goqZtyALigB1VnAAmJAO5QAPlAA3ElZ3TkIbVG1HKAgATwAHMBIYKHo6QQBDEAAVezDw6UlreABGJ3QscXDw0goqMFoGO3Z8gi4AGmK4ACZyjERsHChJSShAOzNALASYAFc4IghWMkB75UBTuU0o-UBTc0ARyMB-BMAzaMApFWtwgCdIadOEGbmFskj7J1cPbz9A4NCq6u-ayho6RhRVrVHjhDpdADMThu80WCAqeG+BHOEEu8P6gxOSN+9UagLyWLa1h4PGA8AqIygXjcni8wCEwmopQpo2pLzpDKZPXQLKpNO89JEXIhPIwlIZgsZ8Ah1CIWRAKDZtLQvKVAqAA)里会出现两处报错
- [x] **noImplicitThis** *(Enable error reporting when `this` is given the type `any`.)*  

```ts
let fn = () => {
  // 报错: The containing arrow function captures the global value of 'this'
  console.log(this)
}

let obj = {
  /**
   * 这里指定this类型似乎没有用, 还是会报错
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


## 泛型

##### **泛型基础语法**

```ts
// 使用字面量的方式定义泛型类型的函数
const fn1: <T>(arg: T) => T = arg => arg
function fn2<T>(arg: T): T {
  return arg
}

// 使用类型别名定义泛型类型的函数
type FnType1 = <T>(arg: T) => T
type FnType2 = {
  <T>(arg: T): T
}

// 使用接口定义泛型类型的函数
interface FnType3 {
  <T>(arg: T): T
}

// 编译器会自动根据传入的类型去推断
let num = fn1(999) // num: number

// 当编译器无法推断时, 使用尖括号显式的指定其类型
let str = fn1<string>('hello') // str: string
```

##### **泛型约束条件**

```ts
// 限制传入的类型里必须包含 length 属性, 并且是 number 类型
function fn<T extends { length: number }>(arg: T): number {
  return arg.length
}

fn('hello') // 正确
fn([1, 2, 3]) // 正确
fn({ length: 0 }) // 正确
fn(999) // 报错: number 类型不能分配给 { length: number }
```

```ts
// keyof T 实际上就是在获取obj上的所有key, 然后转成联合类型
function getVal<T, K extends keyof T>(obj: T, key: K) {
  return obj[key]
}

const obj = { name: 'jack', age: 99 }
getVal(obj, 'name') // 'jack'
getVal(obj, 'age') // 99
getVal(obj, 'sex') // 报错: 联合类型 'name' | 'age' 中不包含 'sex'
```

## 类型运算符

### keyof 运算符

`keyof`运算符可以获取到对象类型所有 key 的文字类型或联合类型

```ts
type Point = { x: number, y: number }
type PointKeys = keyof Point // 'x' | 'y'

type Arr = { [key: number]: any }
type ArrKeys = keyof Arr // number

type HashMap = { [key: string]: any }
type HashMapKeys = keyof HashMap // string | number
```

### typeof 运算符

js里的`typeof`和ts里的`typeof`功能相似, 都是获取一个变量的类型

!> ts里的`typeof`只能作用于变量, 直接`typeof 'hello'`是不允许的

```ts
type ErrType = typeof 'hello' // 报错

let str = 'world'
type Str = typeof str // string

let obj = { name: 'jack', age: 20 }
type Obj = typeof obj // { name: string, age: number }


// 获取函数的返回值类型
const fn = () => ({ x: 1, y: 1 })
type Result = ReturnType<typeof fn> // { x: number, y: number }
```

## 索引访问类型

```ts
// 通过索引获取对应属性的类型
type Obj = {
  name: string,
  age: number,
  alive: boolean
}
type AgeType = Obj['age'] // number
type NameOrAge = Obj['name' | 'age'] // string | number
type ValueType = Obj[keyof Obj] // string | number | boolean
type AnyType = Obj['xxx'] // 报错: Obj 上不存在 xxx 属性


// 通过索引获取数组元素的类型
const arr = [
  { name: 'jack', sex: 0 },
  { name: 'ben', sex: 1 }
]
type ItemType = typeof arr[number] // { name: string, sex: number }
type NameType = typeof arr[number]['name'] // string
type sexType = ItemType['sex'] // number

// 通过变量的类型 来获取类型
const key = 'sex'
type keyType = ItemType[typeof key] // number
```

!> 通过索引访问类型时候, 不能使用字符串类型的变量, 但是可以通过变量推导出`文字类型`使用

```ts
const arr = [
  { name: 'jack', sex: '男' },
  { name: 'ben', age: 19 },
  { fn: () => true }
]
arr.forEach(item => {
  type ItemType = typeof item
  (Object.keys(item) as (keyof ItemType)[]).forEach(key => {
    type ValType = ItemType[typeof key] // 这里的 [typeof key] 是文字类型
    let val: ValType = item[key]
  })
})
```


## 条件类型

?> `TypeScript 2.8 新增: 条件类型`, 使得在ts里也可以通过逻辑判断来决定返回的类型, `T extends U ? X : Y`是在判断类型T是否能分配给类型U, 如果可以则返回类型X, 反之则返回类型Y


```ts
interface IdLabel { id: number }
interface NameLabel { name: string }


// 通过函数重载实现方法
function fn1(id: number): IdLabel
function fn1(name: string): NameLabel
function fn1(nameOrId: number | string): IdLabel | NameLabel {
  throw {}
}
fn1('hello') // NameLabel
fn1(999) // IdLabel


/**
 * 通过泛型实现方法
 * <T extends string | number> 首先约束参数的类型只能是字符串或数字
 * (nameOrId: T) 然后指定传入的参数就是T, 方便进行自动推断
 * (T extends number ? IdLabel : NameLabel) 如果T可以分配给number, 那就可以返回`IdLabel`类型
 */
function fn2<T extends string | number>(nameOrId: T): (T extends number ? IdLabel : NameLabel) {
  throw {}
}
fn2('hello') // NameLabel
fn2(999) // IdLabel
```

### 分布式特性

在下面的示例中出现了奇怪的一幕, 类型A和类型B居然不一样, 按理说他们返回的都是`T[]`, 理论上返回值也应该一致才对

```ts
type ToArray1<T> = T[]
type A = ToArray1<string | number> // A = (string | number)[]

type ToArray2<T> = T extends any ? T[] : never
type B = ToArray2<string | number> // B = string[] | number[]
```

出现这个问题是因为条件判断在遇到联合类型的时候具有分布式特性, 比如上面的`ToArray2`会分两次去判断

- 第一次判断`string`能否分配给`any`, 可以的话返回`string[]`, 否则返回`never`
- 第二次判断`number`能否分配给`any`, 可以的话返回`number[]`, 否则返回`never`
- 然后两次结果合并就变成了`string[] | number[]`

想避免这个问题也很简单, 只需要使用中括号把`泛型T`包裹起来: `[T]`, 他就会把`联合类型`当成一个整体去进行条件判断

```ts
type ToArray3<T> = [T] extends any ? T[] : never
type C = ToArray3<string | number> // C = (string | number)[]
```

上面的`ToArray3`只经历了一次判断

- 第一次判断`string | number`能否分配给`any`, 可以的话返回`(string | number)[]`, 否则返回`never`
- 然后输出结果: `(string | number)[]`

很多高级类型也是借助于这个特性来实现的, 比如`Diff<T, U>`和`Filter<T, U>`

### infer 关键词


获取数组`item 元素`的类型, 并且 `GetItemType1` 和 `GetItemType2` 等价


```ts
type GetItemType1<T> = T extends any[] ? T[number] : T
type GetItemType2<T> = T extends Array<infer U> ? U : T


type a = GetItemType1<number[]> // a: number
type b = GetItemType2<number[]> // b: number
type c = GetItemType1<any[]> // c: any
type d = GetItemType2<any[]> // d: any
type e = GetItemType1<Array<string | number>> // e: string | number
type f = GetItemType2<Array<string | number>> // f: string | number
```


#### 实现 ReturnType 高级类型


```ts
// 提取出函数的返回值类型
type GetReturn<T> = T extends (...args: never[]) => infer U ? U : never
type d = GetReturn<() => void> // d: void
type e = GetReturn<() => number[]> // e: number[]


// 定义一个函数重载
declare function fn(x: string): string
declare function fn(x: number): number
declare function fn(x: string | number): string | number


type f = GetReturn<typeof fn> // f: string | number
type g = ReturnType<typeof fn> // g: string | number
```


#### 两个疑问点


```ts
// 为什么这里使用了 never[] 作为函数的参数集合类型
type GetReturnType1<T> = T extends (...args: never[]) => infer U ? U : never
// 我认为我使用 unknown[] 会更合理一些? 因为我根本就不知道别人要传什么
type GetReturnType2<T> = T extends (...args: unknown[]) => infer U ? U : never


// 随便写一个函数, 测试 GetReturnType
const fn = (arg: string | number) => typeof arg === 'string' ? '1' : 1


// 然后我发现


// 使用 never[] 的可以正确获取类型
type FnReturnType1 = GetReturnType1<typeof fn>


// 但是使用 unknown[] 却没有匹配上?
type FnReturnType2 = GetReturnType2<typeof fn>


/**
 * 疑问1: never[] 表示什么意思, 如果说 never 表示无法达到, 那为啥还会有 never[], 无法达到的数组?
 */


/**
 * 疑问2: 为啥 unknow[] 没有匹配成功进入了 false, 按理说应该可以?
 */
```