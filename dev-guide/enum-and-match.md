## 枚举类型

枚举类型列举了该类型的所有可能取值。

定义 enum 类型时，需要列举所有的可能取值，这些值也被称为 enum 的构造器 constructor。构造器之间使用`|`分隔，首个构造器前的`|`可有可无，构造器可以同名，但是参数得不同。

```
enum Color {
  | Red | Blue | Red(Int64, Int64)
}
```

和 struct 一样，enum 只能定义在源文件顶层作用域。

enum 中支持定义成员函数，静态成员函数，成员属性，静态成员属性，操作符重载函数。

```
enum Color {
  | Red | Blue | Red(Int64, Int64)
  func hello() {
    print('hello')
  }
  static func staticHello() {
    print('hello')
  }
  prop a: Int64 {
    get() {
      10
    }
  }
  operator func +(a: Color) {
  }
  static prop staticA: Int64 {
    get() {
      10
    }
  }
}
Color.Red.hello()
Color.staticHello()
print(Color.Red.a)
print(Color.staticA)
```

enum 支持递归定义

```
enum Tree {
    | Empty | Leaf(Int64) | Node(Int64, Tree, Tree)
    func traverse(): Unit {
        match (this) {
            case Empty => ()
            case Leaf(value) => print(value)
            case Node(value, left, right) =>
            print(value)
            left.traverse()
            right.traverse()
        }
    }
}
```

在 enum 调用的上下文中不存在命名冲突的时候，调用 enum 时可以省略类型名，如果有冲突的话，就必须加上类型名。

```
enum Color {
  | Red | Blue
}
let red = Red
let blue = Blue
```

## Option

Option 是一个泛型 enum，表示一个类型的取值可能为有值和无值。

```
enum Option<T> {
  | Some(T)
  | None
}
```

它支持`:?`语法糖，可以快速定义一个 Option 类型

```
let a : ?Int64 = Some(10)
```

当上下文明确知道`Option<T>`的类型时，可以直接为一个 Option 的类型赋值一个 T 类型。

```
let a: Option<Int64> = 10
```

在使用 None 值时，如果上下文类型不明确，需要明确指定类型

```
let none = None<Int64>
```

[上一篇：结构类型](./struct.md)
