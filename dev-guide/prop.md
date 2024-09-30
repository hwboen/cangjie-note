属性就像加强版的普通变量，将变量的访问和修改抽离为对应的 getter/setter 函数，可以实现访问控制，监控记录，跟踪调试、数据绑定的功能。

属性可以在 class，interface，enum，struct，extend 中定义。和变量一样，它也分为可变属性和不可变属性。使用 mut 修饰的属性为可变属性，拥有 setter 函数，否则为不可变属性，只有 getter 函数。

属性可以分为实例成员属性和静态成员属性，也可以在 interface 和 abstart class 中定义抽象属性。

在重写属性的时候，需要保持属性的 mut 属性。

[上一篇：接口](./interface.md)

[下一篇：子类型关系](./sub-type.md)
