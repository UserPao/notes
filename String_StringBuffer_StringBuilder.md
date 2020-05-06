# String

- 被声明为final类，内部变量亦被final修饰
- immutable（不可变）类 
  - 当需要声明多个相同的字符串的时候，只会声明一个

### StringBuffer、StringBuilder

- 实现原理都是基于可修改的char数组，默认大小是16
- 都继承自AbstractStringBuilder
- StringBuffer线程安全（直接用的synchronized 关键字），StringBuilder非线程安全

