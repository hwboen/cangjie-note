class 与 struct 的主要区别在于，class 是引用类型，struct 是值类型，值类型在传递和赋值的时候会进行复制，另外，class 主要用于实现面向对象的模式，类之间可以实现继承，而 struct 之间无法实现继承。

class 的成员变量，静态成员变量，成员属性，静态成员属性，静态初始化器，成员函数，静态成员函数，普通构造函数，主构造函数，操作符函数的特性都与 struct 相同，这里就不再赘述。

class 的访问修饰符除了 protected 稍微有点不同，其他都与 struct 一样。由于 class 之间可以继承，因此被 protected 修饰的变量或函数，除了在相同模块下可见，在子类中也可见。

## class 终结器

class 支持定义终结器，一般用于释放资源。终结器在类的实例被垃圾回收的时候调用。终结器的函数名固定为`~init`，且无参数，无返回值，无任何修饰符。

```
class A() {
  ~init() {
    unsafe {LibC.free(p)}
  }
}
```

使用终结器有以下的限制：

- 无法显示调用
- 拥有终结器的类无法使用 open 修饰符
- 终结器的调用时机时不确定的，就像 gc 的回收时间不固定
- 终结器可能在任意一个线程上运行

## This 类型

class 的实例成员函数返回类型支持 This 类型，用于指代当前实例，可以实现链式调用的效果

```
class A {
  func hello() {
    this
  }
}
```

当在子类中重写父类中的返回类型为 This 的函数时，子类中重写函数的 This 类型指向子类本身。

## 继承

抽象类默认可被继承，即默认有 open 语义，抽象类的成员必须被 public 或 protected 修饰符修饰。

```
abstract class A {
  public func f()
}
```

非抽象类如果需要被继承，需要使用 open 修饰。父类中的实例成员默认不可被重写，如果需要，使用 open 修饰符，前提得满足 class 是 open 的。

```
open class A {
  public open func f() {}
}
class B <: A {
  public func f() {}
}
```

实例成员的 open 也可被继承，也就是说，如果二级子类中没有重写这个成员，那么在三级子类中仍然具有重写的权利。

```
open class A {
  public open func f() {}
}
open class B <: A {}
open class C <: B {
  public func f() {}
}
```

class 只支持单继承，可实现多个接口。当一个 class 没有直接父类的时候，它的父类默认是 Object,Object 是所有类型的父类，没有直接父类，且没有任何成员。

因为继承会直接继承父类的属性和方法，所以多继承会存在菱形问题，例如 B,C 都继承了基类 A,D 同时继承了 B 和 C，A 中有一个方法 f，当 B,C 同时重写了该方法，D 在调用 f 时，无法确定调用 B 中的 f 还是 C 中的 f，产生了所谓的二义性。并且继承反映的是`is-a`的关系，接口反映的是`can-do`的关系，多接口不仅不会产生二义性，并且也让继承关系更加清晰，以职能的形式赋值实现类。

sealed 修饰符可以修饰抽象类和接口，表示该类只能在本包中被继承，蕴含了`public/open`的语义

```
sealed interface A {}
sealed abstract class B {}
```

## 构造函数调用 this，super

`this(args)`和`super(args)`调用时必须为构造函数的第一个表达式，this 调用自身构造函数，super 调用父类构造函数，两者不能同时调用。

子类的主构造函数无法通过 this 调用构造函数

```
open class Father {}
class Child <: Father {
  Child() {
    this(12) // Error
  }
  init(a: Int) {
  }
}
```

子类的构造函数中如果没有显示调用父类构造函数，或者其他构造函数时，编译器会在构造函数中开始的位置插入`super()`，默认调用父类的无参构造函数，如果父类中不存在无参构造函数，就会编译报错。

```
open class Father {
  Father(a: Int) {}
}
class Child <: Father {
  let a: Int
  init() { // Error，父类中不存在无参构造函数
    a = 10
  }
}
```

## 覆盖和重定义

子类可以 override 覆盖父类中使用 open 修饰的成员函数，静态成员函数无需 open 修饰，只要 class 是 open 的，open 无法修饰成员变量。在子类中重写对应的方法时，可选 override 修饰符。

```
open class Father {
  public open func f() {}
}
class Child <: Father {
  override func f() {}
}
```

静态成员函数无需使用 open 修饰符

```
open class Father {
  static func hello() {}
}
class Child <: Father {
  redef static func hello() {}
}
```

如果被覆盖的函数存在命名参数，则子类重定义时命名参数要保持一致。

```
open class Father {
  public open func hello(a !: Int) {
    print(a)
  }
}
class Child <: Father {
  public func hello(a !: Int) {
    print(a)
  }
}
```

当覆盖静态泛型函数时，子类的泛型约束要比父类更宽松或相同，宽松可以简单理解为父类，因为父类有的功能，子类都有，因此也就更宽松，访问修饰符也是一样，子类的访问修饰符需要比父类的更宽松。

```
open class A {}
open class B <: A {}
open class C <: B {}

open class Father {
    public static func f<T>(a: T) where T <: B {
        a
    }
}

class Child <: Father {
    public redef static func f<T>(a: T) where T <: A {
        a
    }
}
```

[上一篇：模式匹配](./match.md)

[下一篇：接口](./interface.md)
