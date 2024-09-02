仓颉支持面向对象，函数式，命令式等多种编程范式，还可以快速设计 DSL

- 类和接口：仓颉支持使用传统的类和接口来实现面向对象范式编程，只允许单继承，每个类只能有一个父类，可以实现多个接口。每个类都是 Object 的子类，所有的仓颉类型包括 Object 都隐式的实现了 Any 接口

  仓颉使用 open 修饰符，来控制一个类能不能被继承，或者一个对象的成员函数能不能被子类重写(override)

  仓颉的 interface 之间也可以继承，并且不受单继承的约束，一个 interface 可以继承多个父 interface

- 函数作为一等公民：仓颉中函数既可以作为普通表达式使用，也可以作为参数传递，函数返回值，被保存在其他数据结构中，或者被赋值给一个变量使用。对象或结构体的成员函数也可以作为一等公民使用。

- 代数数据类型和模式匹配：代数数据类型是一种复合类型，指由其他数据类型组合而成的类型。类型常见的代数数据类型是“和”类型（如 enum）和“积”类型（如 struc, tuple）。“和”可以理解为类型的集合，"积"可以理解为类型的组合。

  ```Cangjie
  enum BinaryTree {
    | Node(Int64, BinaryTree, BinaryTree)
    | Empty
  }
  ```

  使用模式匹配对 enum 实例的值进行解析

  ```
  func sumBinaryTree(bt: BinaryTree): Int64 {
    match (bt) {
        case Node(v, l, r) =>
        v + sumBinaryTree(l) + sumBinaryTree(r)
        case Empty => 0
    }
  }
  ```

- 泛型：将类型抽象并参数化，例如`Array<T>`，泛型带来的好处有：

  - 代码复用：一套逻辑支持多种类型。
  - 类型安全：在支持多种类型的同时保证类型安全。
  - 性能提升：在不使用泛型的时候，需要使用较为通用的类型例如 Object，需要频繁的进行类型转换。使用泛型能够避免非必要的类型转换，提升程序性能。

  还可以使用`where`关键字对泛型中的类型变元给出具体的约束

  ```
  func lookup<T>(element: T, arr: Array<T>): Bool where T <: Equatable<T> {
    for (e in arr) {
        if (element == e) {
            return true
        }
    }
    return false
  }
  ```

  仓颉的泛型不支持协变，协可以理解为协助，一致，即泛型类型参数与子类型间保持一致。如果 B 是 A 的子类型，那么 `Array<B>`也可以视为 `Array<A>`的子类型，这在仓颉中是不支持的，因为这可以避免协变导致的类型不安全问题。

  ```
  open class Fruit {}
  class Apple <: Fruit {}
  main() {
    var a: Array<Fruit> = []
    var b: Array<Apple> = []
    // a = b 编译报错
    // b = a 编译报错
  }
  ```

仓颉支持类型扩展，允许我们在不修改原有代码的情况下，对已有的代码做如下的扩展：

- 添加函数
  ```
  extend String {
    func printSize() {
        print(this.size)
    }
  }
  main() {
    "123".printSize()
  }
  ```
- 添加属性
- 添加操作符重载
- 实现接口

  ```
  sealed interface Integer {}
  extend Int8 <: Integer {}
  extend Int16 <: Integer {}
  extend Int32 <: Integer {}
  extend Int64 <: Integer {}
  let a: Integer = 32
  ```

仓颉支持命名参数，可以减少参数的顺序依赖性

```
func a(name!: String, age!: Int32 = 12) {
    name + age.toString()
}
```

仓颉还支持尾随 lambda (trailing lambda)语法糖，如果函数的最后一个形参是函数类型，那么在调用函数的时候，可以提供一个 lambda 表达式，当 lambda 表达式的为无参函数时，还允许省略 =>

```
func unless(condition: Bool, f: () -> Unit) {
    if (!condition) {
        f()
    }
}
main() {
    unless(true) {
        print('traling lambda')
    }
}
```

管道操作符（Pipeline）：仓颉中为了简化嵌套函数调用的语法，引入了 Pipeline 的语法糖，来更直观的表达数据流向。

```
func double(x: Int64) {
    x * 2
}

func increment(x: Int64) {
    x + 1
}

main() {
    let a = double(increment(double(double(5))))
    let b = 5 |> double |> increment |> double |> double
}
```

仓颉支持操作符重载，可以作用在开发者自己的类型上

```
struct Point {
    let x: Int
    let y: Int
    init(x: Int, y: Int) {
        this.x = x
        this.y = y
    }
    operator func +(r: Point) {
        Point(this.x + r.x, this.y + r.y)
    }
}
main() {
    let a = Point(1, 2)
    let b = Point(3,4)
    let c = a + b
}
```

仓颉提供了属性（Prop），它可以将成员变量的访问封装为`getter`,`setter`的访问形式

```
class Point {
    private let _x: Int
    private var _y: Int
    init(x: Int, y: Int) {
        _x = x
        _y = y
    }
    prop x: Int {
        get() {
            return _x
        }
    }
    mut prop y: Int {
        get() {
            return _y
        }
        set(y) {
            _y = y
        }
    }
}
```
