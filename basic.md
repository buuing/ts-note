
# TypeScript 基础类型
---

## String [字符串]

```ts
let name: string = 'bob' // 字符串

let info: string = `my name is ${name}` // 模板字符串
```

## Number [数字]

```ts
let decLiteral: number = 6 // 十进制

let hexLiteral: number = 0xf00d // 十六进制

let binaryLiteral: number = 0b1010 // 二进制 - es6新增

let octalLiteral: number = 0o744 // 八进制 - es6新增

let notNumber: number = NaN

let num: number = Infinity
```

## Boolean [布尔]

```ts
let isDone: boolean = false
```


## Array [数组]

- `普通数组`: 可以使用`T[]`或`Array<T>`这两种方式

```ts
// 数字类型的数组
let num1: number[] = [1, 2, 3]
let num2: Array<number> = [1, 2, 3]

// 字符串类型的数组
let list: string[] = ['1', '2', '3']

// 任意类型的数组
let arr: any[] = [1, '2', undefined, null]
```

- `只读数组`: `readonly T[]`或`ReadonlyArray<T>`两种语法是等价的

```ts
// 只读类型的数组
let arr1: readonly number[] = [1, 2, 3]
let arr2: ReadonlyArray<number> = [1, 2, 3]

arr1 = [] // 正确
arr2 = [] // 正确
arr1[0] = 999 // 报错: 'readonly number[]' 类型的索引签名只允许读取
arr2[0] = 999 // 报错: 'readonly number[]' 类型的索引签名只允许读取


// ReadonlyArray 类型的数据不能赋值给 Array 类型, 但是反之可以
let arr3: ReadonlyArray<number> = [] as const // 正确
let arr4: ReadonlyArray<number> = [] as Array<number> // 正确
let arr5: Array<number> = [] as ReadonlyArray<number> // 报错: 只读类型的数组不能分配给 'number[]' 可变类型
```

## Tuple [元组]

元组与数组类似, 但是元组需要清楚的知道长度, 以及每一项元素的具体类型 (各元素的类型不必相同)

```ts
let arr1: [string, number] = ['jack', 18]

// 当访问指定索引的元素时, 可以得知确切的类型
arr1[0].toLowerCase() // 字符串
arr1[1].toFixed(2) // 数字


// 元组也同样可以设置可选值
let arr2: [string, number?, string?]
arr2 = ['name']
arr2 = ['ben', 10]
arr2 = ['jack', 20, '男']
arr2 = ['any', undefined, '保密']
arr2[1] // number | undefined
arr2.length // 1 | 2 | 3


// 元组也可以有剩余元素
let arr3: [...string[], number] = ['hello', 'world', 999]
let arr4: [string, number, ...boolean[]] = ['', 0, true, true]


// 只读类型的元组
let arr5: readonly [string, number]
arr5 = ['jack', 10]
arr5 = ['jack', 10] as const
arr5[0] = 'ben' // 报错: 不能给只读属性赋值
```

## Enum [枚举]

> 枚举类型是在运行时真正存在的对象 *(除了常量枚举)* 

##### **常量枚举**

在`enum`前面加上`const`关键字可以形成常量枚举, 常量枚举的好处是在编译时不会生成对应的js代码, 而是会直接把所有使用常量枚举的地方替换成结果, 可以减少内存的占用以及代码的体积

```ts
const enum Letter {
  a = 'A',
  b = 'B'
}
```

##### **数字枚举**

枚举类型是基于普通对象类型的扩展类型, 如果没有指定他的值, 则默认会从0开始++

```ts
enum Letter1 { a, b }
console.log(Letter1.a) // 0
console.log(Letter1.b) // 1
console.log(Letter1[0]) // a
console.log(Letter1[1]) // b

enum Letter2 { a = 10, b }
console.log(Letter2.a) // 10
console.log(Letter2.b) // 11
console.log(Letter2[10]) // a
console.log(Letter2[11]) // b
```

##### **字符串枚举**

但是只有数字枚举具有反向映射的特性, 字符串枚举则不会生成该效果

```ts
enum Letter {
  a = 'A',
  b = 'B'
}

console.log(Letter.a) // A
console.log(Letter.A) // 报错
```

##### **异构枚举**

异构枚举似乎没有什么实际的应用场景

```ts
enum Letter {
  a = 1,
  b = 'B',
}
```

## Any

在第三方代码库未提供`.d.ts`声明文件的情况下只能使用`any`来跳过类型检查, 并在上面随意调用方法

```ts
let notSure: any = '1'
notSure = true
notSure()

let arr: any[] = [1, '2', true]
```



## Void

> 如果开启了[**`strictNullChecks`**](https://www.typescriptlang.org/tsconfig#strictNullChecks)选项 (默认为false), `void`类型的变量就不能设置为`null`了
> - [x] strictNullChecks *(When type checking, take into account `null` and `undefined`.)*

```ts
// void 类型只能赋值 undefined 或 null
let n: void = null
let u: void = undefined

// 当函数没有返回值的时候, 可以使用 void 类型
function fn1(): void {
  console.log('hello world')
}

// 或者是当 Promise 没有返回值的时候, 也可以使用 void 类型
function fn2(): Promise<void> {
  return new Promise(resolve => {
    resolve()
  })
}
const res: Promise<void> = fn2()
```

!> **但是当函数的返回值是`void`类型时, 会产生一些意料之外的效果**

```ts
// 当 void 返回值定义在变量上, 函数并不会强制你返回空值或者不返回, 只是起到了一个忽略返回值的作用
const fn1: () => void = () => {
    return true // 正确
}
const fn2: () => void = function () {
    return true // 正确
}

// 当 void 返回值直接定义在函数体上
const fn3 = (): void => {
    return true // 报错: 不能把 boolean 类型分配给 void 类型
}
const fn4 = function (): void {
    return true // 报错: 不能把 boolean 类型分配给 void 类型
}

// 并且你会发现所有函数的返回值都是 void 类型
const res1 = fn1() // void
const res2 = fn2() // void
const res3 = fn3() // void
const res4 = fn4() // void
```

## Null

```ts
let n: null = null
```

## Undefined

```ts
let u: undefined = undefined
```

## Never

`never`表示没有任何类型, 一般用来表示函数永远也无法到达尾部, 可能中途抛出了错误, 也可能就是个死循环, 或者是递归没有设置终止条件

```ts
// 报错
function error(): never {
  throw new Error()
}

// 无限递归
function dfs(): never {
  return dfs()
}

// 死循环
function loop(): never {
  while (true) {}
}

// 当联合类型中没有任何剩余时也会出现
function fn(x: string | number) {
  if (typeof x === 'string') {
    // ...
  } else if (typeof x === 'number') {
    // ...
  } else {
    x // never类型, 永远也不会走到这里
  }
}
```

## Unknow

`unknown`类型可以用来表示任何值, 类似`any`类型, 但是比`any`类型更加安全

```ts
let noType: unknown
noType = 'hello'
noType = 999
noType = [1, 2, 3]
noType = { name: 'jack' }
noType = null
noType = undefined
noType = () => {}

noType() // 报错, unknown类型不能执行任何操作

function fn1(obj: unknown) {
  obj.fn() // 报错, unknown类型不能执行任何操作
}
function fn2(obj: any) {
  obj.fn() // 正确
}
```

<!-- ## Object

特殊类型`object`是指任何的非原始类型*(string、number、bigint、boolean、symbol、null、 或undefined)*, 不等同与`{}`空对象类型, 也不等同于全局类型`Object`

- 宽泛的`object`类型

```ts
let obj: object = {
  name: 'jack'
}

let newObj: object = Object.create(obj)
```

- 精确的指定有哪些属性

```ts
// 属性结尾可以使用`,`或`;`或什么都不加
let obj: {
  name: string
  age: number
  sex?: string
}

obj = {
  name: 'jack',
  age: 10,
  sex: '男',
  info: '' // 报错, 因为前面的类型里没有定义 info 属性
}
``` -->

## 断言

### 类型断言

在某些情况下, ts无法正确的推断出变量的类型, 此时需要`类型断言`来准确的告知ts变量的类型

!> `类型断言`有两种写法: `<T>`或`as T`, 但是在`jsx`语法下只有`as T`可以生效, 因为尖括号会被误认为是标签

##### **基础用法**

```ts
let str: unknown = ''
// 使用`<T>`
;(<string>str).length
// 使用`as T`
;(str as string).length
```

##### **常见场景**

```ts
let data = {
  name: 'jack',
  age: 10
}

// 如果不加断言, ts 只能推断出 key 仅仅是一个 string 类型
Object.keys(data).forEach(key => {
  data[key] // 报错, 这里 ts 认为 key 是任意字符串, 比如 data['abc']
})

// 加了断言之后, key 就变成了 'name' | 'age' 联合类型
;(Object.keys(data) as (keyof typeof data)[]).forEach(key => {
  data[key] // 正确, 这里的 key 只能是 data 里存在的键
})
```

### 非空断言

```ts
// 如果你十分肯定div标签一定存在, 可以使用非空断言, 因为 querySelector 的返回值可能是 null
let box1: HTMLDivElement | null = document.querySelector('div')
box1!.innerHTML

// 或者使用类型断言, 来告诉 ts 这里一定不是 null
let box2 = document.querySelector('div') as HTMLDivElement
box2.innerHTML
```

### 常量断言

> `TypeScript 3.4 新增`

一般情况下 ts 会认为数组是可变的, 所以在某些情况下传入一个长度可变的数组, 就会导致一些奇怪的报错

```ts
// 函数 fn 只接受两个参数
const fn = ([x, y]: readonly [number, number]) => {}

const arr = [1, 1] // 假设我现在有一个数组arr, 但是传参的时候发现报错了
fn(arr) // 报错: 目标需要2个元素，但arr的元素可能较少

/**
 * 这很奇怪, 我上面明明给 arr 定义的就是两个参数, 为什么 ts 会觉得我的参数长度不正确呢
 */

// 解决方案1: 直接传入一个长度正好的数组, 而不是变量
fn([1, 1]) // 正确

// 解决方案2: 传入一个元组类型的变量, 因为元组类型长度是确定的
const arr2: [number, number] = [1, 1]
fn(arr2) // 正确

// 解决方案3: 传入一个只读数组, 因为只读数组是不可被改变的
const arr3 = [1, 1] as const
fn(arr3) // 正确
```