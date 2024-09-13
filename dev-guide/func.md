# 函数

## 定义函数

```
func add(a!: Int64 = 1, b: Int64) {
  a + b
}
```

函数的参数可以分为非命名参数和命名参数，只有命名参数可以设置默认值，非命名参数需要位于命名参数之前。

函数体中所有 return 语句的类型或者函数体的最后一项表达式为最终函数的类型。

## 调用函数

实参：调用函数时传入的参数。

```
func add(a: Int64, b!: Int64) {
  a + b
}
add(1, b: 3)
```

## 函数类型

在仓颉中，函数是一等公民(first-class citizens)，可以作为函数的参数、返回类型，也可以赋值给变量。

函数本身也有类型，称为函数类型，`(Int64, Int64) -> Int64`，它由参数类型与函数的返回类型构成。

也可以给函数类型的参数命名，以便代码更直观清晰，但是一旦命名，就需要给所有的参数命名：

```
let sayHello: (name: String, age: Int) -> Unit = {name: String, age: Int => print(name + age.toString())}
```

函数可以作为函数的参数也可以作为函数的返回值：

```
func add(a: Int, b: Int) {
  a + b
}
func returnAdd(add: (a: Int, b: Int) -> Int) {
  add
}
```

当函数被重载时，赋值给变量时需要明确函数的类型。

## Lambda

Lambda 来源于数学和计算机科技的 Lambda 演算，它是一套用于研究函数定义，调用，递归的形式化系统。

Lambda 的形式定义如下：

```
{p1: T, ..., pn: Tn => expressions | declarations}
```

`=>`之前的为参数列表，之后的为 Lambda 表达式体，参数列表和表达式体都可以为空，例如`{=>}`，它的函数类型为`() -> Unit`。

Lambda 的返回类型是根据表达式体自动推演出来的，如果在编译阶段根据上下文推断不出 Lambda 的类型会报错。

如果上下文可以明确的推断出 Lambda 的函数类型，则可以省略 Lambda 参数列表中的类型信息。

`=>`只有在参数列表为空，并且支持`尾随Lambda`的场景下才能省略。

Lambda 表达式支持立即调用

```
let a = {a:Int, b: Int => a + b}(4,5)
```

## 闭包 Closure

一个函数或 Lambda 从定义它的静态作用域中捕获了变量，这个函数或 Lambda 和这个捕获的变量一起称为一个闭包。

以下几种场景称为变量捕获：

- 函数的参数缺省值中访问了本函数之外的局部变量，注意是局部变量。

  ```
  func outer() {
    let outerB = 64
    func closure(a!: Int = outerB) {
      a
    }
  }
  ```

- 函数或 Lambda 内访问定义在本函数或本 Lambda 之外的局部变量，注意是局部变量。

  ```
  func outer() {
    let outerB = 64
    func closure(a!: Int) {
      a + outerB
    }
    let closureLambda = { => outerB }
  }
  ```

- class/struct 内的非成员函数或 Lambda 访问了实例成员变量或 this

  ```
  class OuterClass {
    let a = 64
    func classFunc() {
      func closure() {
        a
      }
      let lambdaClosure = { => this.a}
    }
  }
  ```

只有以上三种变量的访问属于变量捕获，以下这些场景没有发生变量捕获：

- 访问本函数或本 lambda 内的局部变量
- 访问本函数或本 lambda 的形参
- 访问全局变量或静态成员变量
- 实例成员函数或属性访问本实例成员变量。因为成员函数或属性默认拥有 this 参数，通过 this 来访问成员变量。

变量捕获一些隐藏的规则：

- 变量捕获发生在闭包定义时
- 被捕获的变量必须在闭包定义时可见，不能在闭包代码后面定义
  ```
  func outer() {
    func closure() {
      a
    }
    let a = 64 // Error
  }
  ```
- 被捕获的变量必须在闭包定义前完成初始化
  ```
  func outer() {
    let a: Int
    func closure() {
      a
    }
    a = 64 // Error
  }
  ```
- 捕获的变量如果是引用类型，则仅仅捕获引用值

闭包让我们可以在函数外部访问和修改函数内部的状态，但是这种能力如果不加限制，会导致闭包会传递到意外的上下文中，造成所谓的“闭包逃逸”的问题，这里的逃逸就可以简单理解为闭包被传递到意外的上下文中，会导致函数内部的状态被意外的修改，或者造成循环引用的问题。闭包逃逸在捕获可变变量 var 的时候，会更严重，因此对于 var 可变变量的闭包，仓颉做了传递的限制，这类闭包只能被调用，无法作为一等公民使用，即无法赋值给变量或作为函数的形参或作为函数的返回值，以防止进一步的传递扩散。

存在一种很隐蔽的闭包逃逸传递，就是一个函数调用了一个闭包，但是闭包捕获的可变变量 var 不在该调用函数的作用域中，那么这个调用闭包的函数就被传递，也相当于具备了可变变量闭包的特性，也不能作为一等公民使用。

```
func outer() {
  var a = 64
  func closureOuter() {
    func closure() {
      a
    }
    closure()
  }
  closureOuter // Error
}
```

如果闭包捕获的可变变量 var 位于直接父级函数中，那么即使在这个父级函数中调用了闭包，父级函数不受影响，仍然可以正常作为一等公民使用。因为这里的可变变量不具备传递性，不会造成闭包逃逸。

```
func outer() {
  func f() {
    var a = 64
    func closure() {
      a
    }
    closure()
  }
  f // Ok
}
```

因为静态成员变量和全局变量的访问不属于变量捕获，所以访问可变的 var 静态成员变量或者全局变量的函数仍然可以作为一等公民使用。

## 函数调用语法糖

语法糖是一种程序的简化，它可有可无，但是有，就会像糖一样给你带来良好的编程体验。

- 尾随 Lambda：如果函数最后一个形参是函数类型，并且实参是 lambda 时，就可以采用尾随 Lambda 的语法

  ```
  func test(f: () => Unit) {}
  test {
    print("lambda")
  }
  ```

- Flow 表达式：流水线式调用

  - Pipeline 表达式：数据像在一条管道中按步骤按序一条直线往下流动，前一个表达式的返回值是下一个表达式的参数。当数据需要经过流水线的一步步处理时，可以使用 Pipeline 表达式来简化代码。
    ```
    func step1(a: Int) {
      a + 1
    }
    func step2(a: Int) {
      a + 2
    }
    func step3(a: Int) {
      a + 3
    }
    let pipeline = 1 |> step1 |> step2 |> step3
    ```
  - Composition 表达式：组合多个单参函数组合成一个新的函数，前一个步骤的函数类型需要为下一个函数类型的子类型。

    ```
    func step1(a: Int) {
      a + 1
    }
    func step2(a: Int) {
      a + 2
    }
    func step3(a: Int) {
      a + 3
    }
    let step123 = step1 ~> step2 ~> step3
    let step23 = step2 ~> step3
    ```

    Pipeline 和 Composition 流操作符不能使用在有命名参数的函数类型上，如需使用，可以借助 lambda 构建一个新的函数

    ```
    func step1(a!: Int) {
      a + 1
    }
    func step2(a: Int) {
      a + 2
    }
    let compos = {a: Int => step1(a: a)} ~> step2
    ```

    函数如果存在非命名参数，那么当它的命名参数有默认值的时候，也可以使用流操作符

    ```
    func step2(a: Int, b!:Int = 30) {
      a + b
    }
    let b = 1 |> step2
    ```

- 变长参数，当函数形参最后一个参数为 非命名 Array 类型时，调用该函数传入对应数组实参的时候可以直接传入序列

  ```
  func f (a: Int, b: Array[String]) {
  }
  f(1, "a", "b", "c")
  ```

## 函数重载

一个作用域中，同一个函数名对应多个函数定义，并且函数的参数类型或个数不同，这种现象称为函数重载。

- 当同名泛型函数的形参个数与类型不同时，构成函数重载，如果形参类型为泛型时，则比较泛型参数的定义顺序。

```
func f<X, Y>(a: X, b: Y) {}
func f<X, Y>(a: Y, b: X) {}
```

- 类型变元的约束不构成重载

```
interface I1 {}
interface I2 {}
func f<T>(a: T) where T <: I1 {}
func f<T>(a: T) where T <: I2 {} // Error 泛型约束不构成重载
```

构造函数参数不同，包括主构造函数，构成函数重载，主构造函数与 init 函数默认认为具有相同的函数名。

如果在一个作用域可见的范围内存在同名函数，则这些同名函数构成函数重载。（前提是满足函数重载）。例如一个函数定义在外部作用域，它在内部作用域是可见的，在内部作用域中定义的同名函数构成函数重载。

如果子类中存在与父类同名的函数，并且他们的参数类型不同，则构成函数重载。

函数重载只存在于函数定义中，其他的函数定义方式不构成函数重载，不构成函数重载的函数不能定义在同一作用域中。

```
let a: (Int) -> Int = {a => a}
let a: (Float64) -> Float64 = {a => a} // Error
func a(b: Int): Int { // Error
  a
}
```

静态成员函数与实例成员函数不能同名，不能构成函数重载。

```
class A {
  static func hello() {
    print('hello')
  }
  func hello(a: Int) {a} // Error
}
```

函数重载决议规则：

- 高优先级作用域优先，作用域越内层，优先级越高。
- 当作用域优先级相同时，最匹配的函数定义优先。
- 父类和子类的作用域视为同一作用域，规则同第二条。

## 操作符重载

在仓颉中，可以对特定的操作符进行重载，使得原本不支持该操作符的类型，也可以使用该操作符的语义。

操作符函数定义与普通函数定义类似，只不过需要使用`operator`关键字，函数名为对应的操作符。

```
class Point {
    let x: Int
    let y: Int
    Point(x: Int, y: Int) {
        this.x = x
        this.y = y
    }
    operator func +(p: Point) {
        Point(this.x + p.x, this.y + p.y)
    }
}
```

操作符函数具有实例对象的语义，因此不能使用 static 修饰符。只能定义在 class，struct，enum，extend 中。

操作符函数不能为泛型函数，因为它最重要的目的是扩展原有算术操作符的语义，因此不能为泛型函数。

重载操作符函数要求保存原有操作符的语音，一元操作符，例如`-1`，操作符函数没有参数，二元操作符，例如`a + b`，操作符函数有且只有一个参数。

索引操作符(`[]`)的重载比较特殊，它分为赋值和取值两个逻辑，并通过命名参数`value`来区分。

- 赋值操作符函数重载
  ```
  operator func[](arg1: Int, arg2: String, value!: String) {
  }
  ```
- 取值操作符函数重载：参数个数不限制，不能有命名参数
  ```
  operator func [](arg1: Int, arg2: String) {
  }
  ```

索引操作符只有符合索引语义的结构才可重载，除了`enum`，其他不可变类型不支持重载索引操作符。

函数调用操作符`()`函数，参数和返回类型可以是随意的，不能通过`this`、`super`的方式调用，当在 enum 类型上重载函数调用操作符时，优先匹配枚举构造器函数。

如果一个类型已经拓展了操作符函数，便不可再通过 extend 为其添加对应的操作符函数。

## const 函数和常量求值

const 表达式可以在编译时求值，减少运行时计算，如果编译时无法求值，就当做普通表达式处理。

```
const func add(a: Int, b: Int) {
  a + b
}
add(1, 2) // 常量表达式，可以在编译时求值
let a = getA()
let b = getB()
add(getA(), getB()) // 当做普通表达式处理
```

const 表达式可以在编译时求值，不依赖运行时，例如不显示构造对象，变量不可变，如果一个表达式显示调用了运行时的特性，例如显示调用构造对象，则无法使用 const 修饰。

```
const a = 64 // Ok
const a = Int(64) // Error
```

对于复合 const 表达式，例如函数表达式，`if else`表达式，要求它们的子表达式都是 const 表达式。

[上一篇：基本类型](./primitive-types.md)
