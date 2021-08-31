
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

可以使用`T[]`或`Array<T>`这两种方式

```ts
// 数字类型的数组
let num1: number[] = [1, 2, 3]
let num2: Array<number> = [1, 2, 3]

// 字符串类型的数组
let list: string[] = ['1', '2', '3']

// 任意类型的数组
let arr: any[] = [1, '2', undefined, null]
```

## Tuple [元组]

元组与数组类似, 元组需要清楚的得知数组的长度, 以及每一项元素的具体类型 (各元素的类型不必相同)

```ts
let arr: [string, number] = ['jack', 18]

// 当访问指定索引的元素时, 可以得知确切的类型
arr[0].toLowerCase() // 字符串
arr[1].toFixed(2) // 数字
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
enum Letter { a, b }

console.log(Letter1.a) // 0
console.log(Letter1.b) // 1
console.log(Letter1[0]) // a
console.log(Letter1[1]) // b
```

```ts
enum Letter { a = 10, b }

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

console.log(Letter3.a) // A
console.log(Letter3.A) // 报错
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

`void`类型只能赋值`undefined`或`null`*(当--strictNullChecks未指定时)*

!> 这里有个坑, 老版本无需指定`strictNullChecks`也是可以直接赋值`null`的

```ts
let param: void = undefined
```

又或者是一个函数没有返回值的时候, 可以使用`void`

```ts
function fn(): void {
  console.log('hello world')
}
```

又或者是`promise`没有返回值的时候

```ts
function fn(): Promise<void> {
  return new Promise(resolve => {
    resolve()
  })
}

// 函数的返回值是 Promise<void> 类型
const f: Promise<void> = fn()
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
```

## Object [对象]

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
```

## 类型断言

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

## 非空断言

```ts
// 如果你十分肯定div标签一定存在, 可以使用非空断言, 因为 querySelector 的返回值可能是 null
let box1: HTMLDivElement | null = document.querySelector('div')
box1!.innerHTML

// 或者使用类型断言, 来告诉 ts 这里一定不是 null
let box2 = document.querySelector('div') as HTMLDivElement
box2.innerHTML
```