# 结构类型

## 定义 struct 类型

struct 主要用于实现用户自定义的值类型，它也可以实现接口。

struct 中可以定义成员变量，静态成员变量，成员属性，静态成员属性，成员函数，静态成员函数，静态初始化器，主构造函数，构造函数。

struct 的成员变量的初值可以延迟到构造函数中。静态成员变量的初值可以延迟到静态初始化器中。

struct 的静态初始化器以 static init 开头，没有函数形参，没有返回值，不能被访问修饰符修饰，并且一个 struct 只能定义一个静态初始化器，不能访问实例成员变量和实例成员函数。

```
struct S {
  static let a: Int64
  static init() {
    a = 64
  }
}
```

struct 支持普通构造函数和主构造函数。普通构造函数通过重载定义可以有多个，函数名为 init：

```
struct S {
  public init(a: Int64) {}
  public init(a: Int64, b: Int64) {}
}
```

主构造函数的函数名和 struct 类型名相同，一个 struc 只能定义一个，支持普通形参和成员变量形参，成员变量形参等同于定义了成员变量。

```
struct S {
  public S(name: String, let age: Int64) {}
}
```

struct 支持成员函数和静态成员函数，静态成员函数不能访问实例成员变量和成员函数，反过来，成员函数可以访问静态成员变量和函数。

```
struct S {
  static let a: Int64 = 30
  public static func hello() {
    print(a)
  }
}
```

struct 成员的访问修饰符有 public,internal,protected, private，默认为 internal。

- public 都可见，对所有模块和包都可见
- protected 同一模块可见
- internal 当前包和子包内可见，包括子包的子包
- private 仅在 struct 定义内可见

struct 不支持递归和互递归定义：

```
struct A {
  let a: A // Error
}
struct B { // Error
  let c: C
}
struct C { // Error
  let b: B
}
```

## 创建 struct 实例

在构造对象的时候，仓颉省略了 new 关键字，创建 struct 实例的时候，只需要调用它的构造函数即可：

```
struct S {
  public init(a: Int64, b: Int64) {}
}
let s = S(1,3)
```

如果要修改 struct 的成员变量，需要满足两个条件，一是定义 struct 的实例变量需要为可变变量，二是 struct 的成员变量也要为可变变量：

```
struct S {
  var a = 64
}
var s = S()
s.a = 100
```

struct 类型具有值语义，在赋值和传递的时候会进行复制，如果它的成员变量是引用类型的话，仅复制引用地址。

## mut 函数

由于 struct 是值类型，它的成员函数无法修改实例本身。为什么呢？因为值类型在传递的时候会进行复制，成员函数访问成员变量涉及了成员变量的复制，也就意味着在值类型的成员函数中的成员变量是经过复制的，是副本，在成员函数中修改这个副本是没有意义的。

可以用过 mut 函数来修改 struct 实例的成员变量，在这个函数内部，this 的语义是特殊的，拥有原地修改值的能力。

```
struct S {
  var a = 10
  mut func add() {
    a += 1
  }
}
```

mut 函数只能在 interface，struct，struct 扩展中定义，class 的成员函数本身就具备修改成员变量的能力，不需要 mut 函数。

当在接口中定义 mut 函数时，struct 实现该接口也需要为 mut 函数，class 则不需要

```
interface I {
  public open mut func add(): Unit
}
struct S <: I {
  var a = 64
  public mut func add() {
    a += 10
  }
}
class C <: I {
  public func add() {}
}
```

struct 的传递即复制，当把 struct 实例对象赋值给接口类型的时候，会对实例对象进行复制：

```
let s = S()
let i: I = s // 复制
i.add()
print(s.a) // 还是原值，不会改变
```

mut 函数无法捕获实例的成员变量，也无法捕获 this。为避免闭包逃逸， mut 函数无法作为一等公民使用和传递，只能被调用。

let 的 struct 变量无法调用 mut 函数，但如果该变量为接口，则可以调用，因为当变量是接口类型时，遵循的就是接口类型的标准。

```
let i: I = S()
i.add()
```

非 mut 实例成员函数无法调用 mut 函数，反之则可以。

[上一篇：函数](./func.md)

[下一篇：枚举和模式匹配](./enum-and-match.md)
