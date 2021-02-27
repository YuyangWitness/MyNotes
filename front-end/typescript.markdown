# Typescript
#### 为什么要学习typescript呢？
- 代码更容易理解了
如果给变量加上类型，那么代码就可以告诉我们需要传入什么类型的东西，也不需要自己打断点或者console.log去获得一个变量的结构。这个其实在写Go的时候深有体会，刚开始学习Go的时候遇到了结构体和类型定义，挺烦的，但是写到后面渐渐就熟悉了也不厌烦了，在查阅一些接口的时候看变量类型就能够知道这是一个什么东西，输入和输出的结构体是什么样。
- 效率更高了
typescript支持可以在不同的代码块和定义中跳转，代码可以自动补全并且有丰富的接口提示
- 更少的错误
编译期间可以发现一些错误，不会出现在运行的时候
- 非常好的包容性
完全兼容javascript

## 基本类型
```
// 五种基本类型
let isDone: boolean = true
let str: string = "str"
let num: number = 123
let getNull: null = null
let getUndefined: undefined = undefined
```
- `undefined`和`null`是所有类型的子类型, 如果`strictNullChecks`设置为false,那么可以写如下代码：
```
let str2: string = undefined
let str3: string = null
```
如果`strictNullChecks`为true，则不行。

- `any`类型表示可以给变量赋任意类型的值，那么any对于一个变量没有任何约束了。
设为any类型的变量就没有代码提示功能，因为系统无法确定它是什么类型，所以无法提示相关的方法。
- 联合类型，一个变量可以赋给多种类型
```
let numOrString: string | number
numOrString = 12
numOrString = "dff"
```

## Array and Tuple
### Array
数组将同类型的数据放在一起
```
// 只能在数组中放入number类型
let array1: number[] = [1,2,3,4,]
```

### Tuple
类数组，可以把不同类型的数据放在一个数组中
定义的时候限制数组中每个索引中有什么类型
```
let tuple1: [number, string] = [1, "12"]
```
如果访问越界，那么会使用联合类型来赋值
```
// 之后添加的元素通过联合类型来判断 string|number
tuple1.push("aa")
tuple1.push("bb")
tuple1.push(123)
```

## Interface
`interface`是用来描述对象的形状(高矮胖瘦？行为？)
对象实现interface需要完全匹配interface定义的需求
```
interface speakable {
  name: string,
  speak(): void
}
let speakMain: speakable = {
  name: "aaa",
  speak(){}
}
```
- 可以使用readonly来描述一个方法或者变量，这样就无法更改定义之后的值了
```
interface speakable {
  readonly name: string,
  speak(): void
}
```
- 也可以在变量或者方法后面加"?"表示是可选项
```
interface speakable {
  name?: string,
  speak?(): void
}
```