# 轻松并发

## 轻量化线程模型

用户态线程，原生协程，拥有独立的执行上下文，并共享内存。用户态线程有有栈和无栈两种实现方式，栈后进先出，天然适应函数的调用链，栈的基本单位是栈帧，栈帧用于保存每个函数的执行状态，包含局部变量、执行位置、返回地址（即该函数执行完后的下一步执行位置）。

- 有栈：仓颉采用有栈的用户态线程模型，每个线程拥有独立的栈来保存当前线程的执行状态。
- 无栈：采用状态机来代替栈，以下用一个简单的例子来说明什么是状态机。

  ```
  class Example:
    def __init__(self):
      self.state = 0

    async def run(self):
      if (self.state === 0):
        print('Step 1')
        // 协程被挂起，保存状态机状态
        await async_operation_1()
        self.state = 1

      if self.state === 1:
        print('Step 2')
        await async_operation_2()
        self.state = 2

      if self.state === 2:
        print('Step 3')
        await async_operation_3()
        self.state = 3
  ```

  无栈模式下，通过标记异步操作点，然后编译器手动插入状态机代码，通过状态机来管理当前线程的执行状态。因此需要引入特殊的语法用来标记异步操作点，最常见的就是`async/await`。
