## 接口定义

接口定义了一套行为的规范，不包含具体的数据和状态。相比继承 is-a，接口强调的是 can-do 是一种行为或能力的抽象。接口中可以定义成员函数，静态成员函数，属性，静态属性，操作符重载函数。interface 默认具有 open 语义，它的成员默认具有 public/open 语义。

接口中可以拥有默认实现。

泛型可以使用接口作为泛型约束：

```
inteface I {
  hello() {}
}
func hello<T>(a: T) where T <: I {
  a.hello()
}
```

sealed interface 表示该接口只能在当前包内被实现，子包中也无法实现，默认具有 public/open 语义。

## 接口继承

一个类可以实现多个接口，之间使用`&`连接，接口也可以继承接口，重写父接口的成员或静态成员时，override 和 redef 关键字是可选的。

## 接口实现

如果接口中的函数返回 class 时，允许实现该接口的子类型在重写该函数时返回子类型。

```
open interface I {
  func hello(): Parent
}
class Parent {}
class Child <: Parent {}
class IClass <: I {
  public func hello(): Child {
    Child()
  }
}
```

## Any 类型

Any 是一个内置的接口类型，所有的接口默认都继承它，所有的类型默认都实现它，因此它是所有类型的付类型，而 Object 是所有类的父类。

```
interface Any {}
```

[上一篇：类](./class.md)

[下一篇：属性](./prop.md)
