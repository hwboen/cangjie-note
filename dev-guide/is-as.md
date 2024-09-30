子类型天然是父类型，不存在类型的转换。在仓颉中类型转换必须显式地进行，不存在隐式转换。

## 数值之间的类型转换

如果数值之间的转换超出数值范围的时候，会在运行时报错

```
let a: Int64 = 10000
let a2 = Int8(a)
```

## Rune 与整数类型的相互转换

Rune 只能与 UInt32 相互转换,转换时超出范围会编译报错或运行时报错

```
let r = r'a'
let a = UInt32(r)
let r2: Rune = Rune(12)
```

## is 和 as 操作符

is 用于判断某个表达式的类型是否是某个类型或者它的字类型

```
print(1 is Int64)
print(1 is String)
open class Parent {}
class Sub <: Parent {}
let sub: Parent = Sub()
print(sub is Parent)
print(sub is Sub)
```

`e as T`，as 用于将某个表达式转化为指定的类型，因为转换会失败，所以 as 表达式的返回类型为`Option<T>`

```
let a = 1 as Int64 // Option<T>Some(1)
let s = 1 as String // None
```

[上一篇：子类型关系](./sub-type.md)
