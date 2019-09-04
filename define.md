## 符号

1. 关系

```
AAA
  // ccc 为 AAA 的成员 类型为 CCC
  ccc: CCC
  // _ddd 为 AAA 的私有成员 如果只定义了公有成员 则可能具有同名加下划线的私有成员
  _ddd: DDD
  // 函数成员 eee 无参数 返回值为 void (可省略)
  eee(): void
  // get 属性
  get fff(): int

  // BBB 为 AAA 的子类
  // 继承放在所有成员定义之后
  < BBB
```

2. 调用

```
// 调用 AAA 类的实例方法 aaa
:AAA.aaa()
  // 缩进代表代码在 aaa 方法内
  // 给 AAA 的成员 a1 赋值 3
  a1 = 3
  // 调用 AAA 的实例方法 aa1
  aa1()
  // 调用成员 b 的方法 bbb, b 为 BBB 的实例
  b:BBB.bbb()
  // 调用 CCC 类的静态方法 ccc
  CCC.ccc()
  // 调用成员 d 的方法 ddd 并将返回结果赋值给 xxx 同时 xxx 为 XXX 类型
  xxx:XXX: = d:DDD:.ddd()
  // 返回 xxx
  xxx
  
```
