---
title: 并发编程 - 原子操作CAS
tags:
  - 并发编程
---

原子操作

## CAS的原理


		利用了现代处理器都支持的CAS的指令，循环这个指令，直到成功为止
		三个运算符：  一个内存地址V，一个期望的值A，一个新值B
		基本思路：如果地址V上的值和期望的值A相等，就给地址V赋给新值B，如果不是，不做任何操作。
		循环（死循环，自旋）里不断的进行CAS操作


## CAS的问题


- ABA问题（添加版本号解决）
	- AtomicMarkableReference，boolean 有没有动过
	- AtomicStampedReference  动过几次
- 开销问题 长期不成功 会一直循环下去
- 只能保证一个共享变量的原子操作

## JDK中相关原子操作类的使用

- 更新基本类型类：AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference
- 更新数组类：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
- 更新引用类型：AtomicReference，AtomicMarkableReference，AtomicStampedReference
- 原子更新字段类：AtomicReferenceFieldUpdater，AtomicIntegerFieldUpdater，AtomicLongFieldUpdater





















