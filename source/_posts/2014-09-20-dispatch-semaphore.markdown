---
layout: post
title: "GCD(semaphore)的日常"
date: 2014-09-20 19:36:00 +0800
comments: true
categories: 
---
 当代码块在执行一些特定操作时，比如异步处理，有时需要等待代码块是否执行完成，这时可以通过`GCD`来帮助我们实现。
### dispatch_semaphore简介
 `dispatch_semaphore_create`创建一个带初始值的semaphore；

 `dispatch_semaphore_signal`增加semaphore的计数，并发送semaphore信号；

 `dispatch_semaphore_wait`减少semaphore的计数，并等待semaphore信号的到达。
 
 示例代码：

    dispatch_semaphore_t sem = dispatch_semaphore_create(0);

    [self singleBlock:^(id param){
        // your codes...
        dispatch_semaphore_signal(sem);
    }];

    dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);

