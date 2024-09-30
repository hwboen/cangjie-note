# 子类型关系

在使用类型 T 的地方，可以使用类型 T 的子类型 SubT，因为子类型是在父类型上的增强，父类型有的功能子类型都有，因此严格来说，子类型 SubT 包含父类型 T。具体来说，函数形参，函数返回值，变量赋值中使用到的父类型的地方，都可以使用子类型。

## 元组类型的子类型

如果元组 A 中的每个类型都是是元组 B 对应位置类型的父类型，那么就可以称元组 B 是元组 A 的子类型。

```
open class C1 {}
class C2 <: C1 {}
open class C3 {}
class C4 <: C3 {}
let a: (C1, C3) = (C2(), C4())
```

## 函数类型的子类型

函数是一等公民，在仓颉中函数也支持子类型。

函数 A 要想成为函数 B 的子类型，首先函数作为整体的表达式，函数 A 的返回值得是函数 B 返回值的子类型。函数 B 作为父类型，它的形参部分在子类型中只能更宽松，只有这样，子函数类型才能完全代替父函数类型。

```
open class R1 {}
class R2 <: R1 {}

open class P1 {}
class P2 <: P1 {}

func subF(p: P1): R2 {}
func parentF(p: P2): R1 {}

func test(f: (P2) -> R1) {}

test(subF) // Ok
test(parentF) // Ok
```

## 永远成立的子类型关系

- 类型 T 永远是自身的子类型即`T<:T`
- `Nothing <: T`，Nothing 是所有类型的子类型
- `T <: Any`，Any 是所有类型的付类型
- `class <: Object`，Object 是所有 class 类型的父类型

[上一篇：属性](./prop.md)
