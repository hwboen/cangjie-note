包是源代码分类管理的工具，同一功能的源代码文件放在一个包中。每个包有自己的命名空间。在仓颉中，包是最小的编译单位。可以将包简单理解为某一个功能的实现。

模块中包含若干个包，可以将它理解为可以独立运行的子系统，一个模块中只能有一个 main 函数入口。模块是第三方开发者发布的最小单位。

## 包的声明

包声明需要位于源文件首行。包名除了必须为普通的标识符，还要求只能包含数字，小写字母和下划线。

包名和源文件目录路径一致，src 为该包的根路径，称为 root 包，root 包名一般与该模块名一致，src 下的目录结构对应该包的子包以及子包的子包。例如目录结构为`src/dir1/dir2`，dir2 目录下文件的包名为`root.dir1.dir2`。同一目录下的源文件包名相同。

子包名不能和当前包的顶层声明同名。例如

```
/src/package_a/a.cj
interface b {}
/src/package_a/b/b.cj // Error
```

## 顶层声明的可见性

顶层声明可以使用可见性修饰符：

- private：仅在当前文件可见
- internal：当前包和子包，包括子包的子包内可见
- protected：当前模块内可见，不同包之间的调用通过 import 引入
- public：模块内外都可见

相同包的声明是共享的，使用时无需导入。

package 支持使用 public,protected,public 修饰符，默认为 public。它控制的是整个包的可见性，但目前测试下来，该包最终的可见性由该包下所有源文件中的包声明的最大访问范围，也就是说你如果你要使用`internal package root.package_a`，则必须使用 internal 修饰该包下所有源文件的包声明，这点很鸡肋。

import 前支持使用所有可见性修饰符，默认为 private。一直没想明白在 import 前使用可见性修饰符的意义是什么，官方也没有一个更详细的解释，就先放着吧。

高可见性的函数在参数和返回值使用低可见性的类型是不合法的：

```
class A {}
public func hello(a: A) { // Error
}
public func hello2(): A { // Error
  A() // Error
}
```

高可见性的变量声明低可见性的类型是不合法的：

```
class A {}
public let a = A() // Error
```

继承时，子类可见性高于父类的可见性也是不合法的，因为理论上子类这个时候已经代表了父类：

```
open class Parent {}
public class Sub <: Parent { // Error
}
interface I {}
public class A <: I {} // Error
```

> 可见性在传递的时候只能变严格，这样才能更可控。

泛型类型的类型实参可见性低于赋值变量的可见性也是不合法的：

```
public class A<T> {}
class B {}
public let a = A<B>() // Error
```

where 约束中的类型上届可见性也不能低于被赋值变量的可见性：

```
interface I {}
public class B<T> where T <: I {} // Error
```

在表达式内部使用低可见性的元素并且不作为返回值时是合法的：

```
class A {}
public func hello() {
  let a = A() // Ok
}
```

函数作为一等公民在传递的时候可以没有可见性的限制，为什么呢？可以简单的理解为在函数传递时又生成了一个新的函数：

```
public let a: () -> Unit = {=>} // Ok
func f() {
}
public func f2(f: () -> Unit) {}
public let f3 = f
```

内置类型诸如 Rune,Int64,String 默认都是 public 的，因为这些类型不是引用类型，在传递的时候是按值传递的，因此可见性对他们没有影响

```
private let a = "hello"
public let b = a // Ok
```

## import

```
package a
import package1.*
import package1.{a,b,c}
import {package1.foo, package2.bar}
```

包的循环导入会在编译时报错。

import 可以被 private, internal, protected, public 修饰，默认是 private import。

当前包的声明优先级高于 import 的声明，如果有重名且不构成重载，则 import 的声明会被当前包覆盖。

core 包是默认隐式导入的。

可以使用 import as 对导入的名字重命名，来解决命名冲突的问题

## 重导出

p2 包大量导入 p1 包中的声明，p2 包可以使用可见性修饰符对引入的 p1 包的声明进行重导出，别的包在引入 p2 包时可以直接使用 p1 包中的声明

- private import：默认为该模式，导入的声明只在当前文件可见
- internal import：导入的内容在当前包和子包，包括子包的子包都可见
- protected import：导入的内容在当前模块内可见
- public import：导入的内容在模块内外都可见

经测试发现，如果重导出的可见性由最开始的声明决定，如果重导出时放大了可见性，会默认忽略采用 private import。
