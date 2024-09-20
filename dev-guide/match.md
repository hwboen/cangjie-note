# 模式匹配

模式匹配故名思义，通过模式来匹配，一个模式可以简单理解为 match 中的一个 case，其中支持的模式有：

- 常量模式：基本类型字面量和 Unit 字面量，可以使用`|`连接多个模式

  ```
  let a = 64
  match(a) {
    case 10 | 20 | 30 => ()
    case _ => ()
  }
  let n = ()
  match(n) {
    case unit => ()
  }
  ```

- 通配符模式 `case _ =>`

- 绑定模式，即将匹配到的值与模式中的标识符进行绑定，但当一个 case 中使用`|`多个模式时，无法使用绑定模式

  ```
  let a = (23, 45)
  match(a) {
    case (a, b) =>
      print(a)
      print(b)
  }
  ```

  绑定模式中的标识符作用域位该 case 语句，且默认为 let

- 类型模式，即通过匹配类型

  ```
  class A {}
  let a = A()
  match(a) {
    case a: A => ()
  }
  ```

- enum 模式

  ```
  enum Color {
    | Red | Blue | Yellow(Int64)
  }
  let c = Color.Red
  match(c) {
    case Red | Blue => ()
    case Yellow(value) => ()
  }
  ```

## 模式的 Refutability 可反驳性

当一个模式即 case 可以匹配所有可能性时，称为 irrefutable 不可反驳的，否则就是 refutable 可反驳的。

```
let a = 30
match(a) {
  case a => () // irrefutable
}
match(a) {
  case _ => () // irrefutable
}
let t = (12, 35)
match(t) {
  case (a, b) => () // irrefutable
}
```

## match 表达式

match 表达式的所有可能性都必须 case 到，否则编译器会报错。

match 表达式支持 pattern guard

```
let t = (1,2)
match(t) {
  case (a, b) where a > 0 => ()
  case _ => ()
}
```

match 表达式支持没有匹配值，直接 case Bool 表达式，注意 match 后面直接跟的是匹配体

```
let a = 12
match {
  case a > 5 => ()
  case _ => ()
}
```

match 表达式的类型如果没有明确的上下文类型，则最后的类型是所有 case 表达式类型的最小公共父类型。

## if-let，while-let 表达式

if-let 和 while-let 表达式原理类似，即满足左侧的模式匹配，则执行对应的分支

```
let r = Option<Int64>.Some(1)
if (let Some(value) <- r) {
  print(value)
} else {
  print(0)
}
while(let Some(value) <- r) {
}
```

## 其他使用到模式匹配

变量定义和 for-in 也支持模式匹配，但是仅支持 irrefutable 的模式

```
enum Color {
  |Red(Int64)
}
let _ =  Red(12)
let Red(red) = Red(12)
for (i in 1..5) {
  print(i)
}
```

[上一篇：枚举](./enum.md)

[下一篇：类](./class.md)
