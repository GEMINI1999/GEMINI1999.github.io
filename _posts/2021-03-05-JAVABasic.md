---
layout: article
title: "Java 基础知识"
key: 11
tags: 
        - Java
        - 编程基础
toc: true
---

## 访问控制符

类内部>本包>子类>外部包

||类内部|本包|子类|外部包|
|-|-|-|-|-|
|public|ok|ok|ok|ok|
|protected|ok|ok|ok|no|
|default|ok|ok|no|no|
|private|ok|no|no|no|

区别：  
public可以被所有其他类访问
private只能被自己本类访问
protected能被自身、同一个包中的类以及子类访问
default能被自身以及同一个包中的类访问
