---
layout: post
title: 'Effective Objective-C 2.0 读书笔记'
author: Q.H.
date: 2018-08-01
cover: 'https://raw.githubusercontent.com/w-qihang/w-qihang.github.io/master/_posts/imgs/effectiveOC.png'
tags: OC iOS 
---

* [熟悉Objective-C](#one)
    * [第2条:在类的头文件中尽量少引用其他头文件](#2)
    * [第3条:多用字面量语法,少用与之等价的方法](#3)
    * [第4条:多用类型常量,少用#define预处理指令](#4)
    * [第5条:用枚举表示状态,选项,状态码](#5)
* [对象,消息,运行期](#two)
* [接口与API设计](#three)
* [协议与分类](#four)
* [内存管理](#five)
* [block与GCD](#six)
* [系统框架](#seven)

<h3 id="one">熟悉Objective-C</h3>
<h4 id="2">第2条:在类的头文件中尽量少引用其他头文件</h4>

* 在不需要知道某个类的实现细节,只需要知道类名的情况下,可在.h中可使用@class类名 来"向前声明"该类,在.m中去import整个.h文件.这样可以尽量降低类之间的耦合,避免增加编译时间;
* 在需要继承某个类或是遵从某个协议,#import是难免的.为了不相互依赖,最好是把协议放在某个单独的头文件中.像"委托协议"就不用放在单独的头文件中,因为协议只有与接受委托的类放在一起定义才有意义.

<h4 id="3">第3条:多用字面量语法,少用与之等价的方法</h4>
使用字面量语法创建字符串,数值,数组,字典会更加简明,不过要注意创建数组或字典时若值中有nil,会抛出异常.通过取下标来访问数组和字典时也要注意nil导致的异常.

<h4 id="4">第4条:多用类型常量,少用#define预处理指令</h4>

* 预处理指令定义的常量不含类型信息,编译器只是会在编译前执行查找和替换,即使重新定义常量值,也不会产生警告;
* 若不公开常量,在实现文件中要同时用static与const来声明;
    * 试图修改const修饰的变量编译器会报错;
    * static修饰符意味着该变量仅在定义该变量的编译单元中可见.
* 需要公开某个常量时,此常量需放在"全局符号表"中,以便可以在定义该常量的编译单元之外使用:
    
    ```
    //In the header file
    extern NSString *const EOCLoginManager; 
    //In the implementation file
    NSString *const EOCLoginManager = @"555";
    ```

<h4 id="5">第5条:用枚举表示状态,选项,状态码</h4>
用NS_ENUM与NS_OPTIONS宏来定义枚举类型,并指明底层数据类型,可确保枚举是用开发者所选的底层数据类型实现出来的,而不会采用编译器所选的类型.

* 表示状态,状态码:

    ```
    typedef NS_ENUM(NSUInteger,EOCConnectionState) {
        EOCConnectionStateDisconnected,
        EOCConnectionStateConnecting,
        EOCConnectionStateConnected,
    };
    ```
* 各选项之间可通过"按位或"来组合,用"按位与"可判断是否已启用某个选项
    ```
    typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
        UIViewAutoresizingNone                 = 0,
        UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
        UIViewAutoresizingFlexibleWidth        = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
        UIViewAutoresizingFlexibleHeight       = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin = 1 << 5
    };

    //组合多个选项
    UIViewAutoresizing resizing = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
        if(resizing & UIViewAutoresizingFlexibleWidth) {
            //UIViewAutoresizingFlexibleWidth is set
        }
    ```


<h3 id="two">对象,消息,运行期</h3>
<h3 id="three">接口与API设计</h3>
<h3 id="four">协议与分类</h3>
<h3 id="five">内存管理</h3>
<h3 id="six">block与GCD</h3>
<h3 id="seven">系统框架</h3>