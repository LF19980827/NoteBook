#### 1、BUG的三种类型——fault，error,failure

- fault：静态存在于软件中的缺陷，如字母拼写出错
- error：存在的错误，如空指针
- failure：已经在可以从外部观测到的失效的行为

#### 2.PIE模型的三个必要条件

PIE模型：使用户或测试人员观测到failure的过程

- Execution/Reachability（执行）：执行时必须通过错误
- Infection（感染）:项目的状态必须是错误的
- Propagation（传播）：错误的中间状态必须传播到最后输出，使得观测到的输出结果和预期结果不一致，即失效。

#### 3.注意

- 有些代码不会触发到fault
- 产生fault的程序，有可能不会产生error
- 有的程序员，有fault和error也有可能不会导致failure