---
title: javaunderlying
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-31 09:05:47
password:
summary:
tags: Java基础
categories: Java
---

### <font color = "green">1. 栈帧中的变量作用域</font>

1. 没有作用域的局部变量不会进入局部变量表。

```java
void test() {
    {
        int val1 = 10;
    }
    int val2 = 20;
}
```

上述代码中，`val1`没有作用域（那个代码块只有它一个，跑完就咩有了），所以不会进入局部变量表中。val2会进入变量表，因为val2作用于test函数。

2. 非静态方法，局部变量表 index=0的位置永远是this指针。静态方法没有this指针。