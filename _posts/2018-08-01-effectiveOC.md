---
layout: post
title: 'Effective Objective-C 2.0 读书笔记'
author: Q.H.
date: 2018-08-01
cover: 'https://raw.githubusercontent.com/w-qihang/w-qihang.github.io/master/_posts/imgs/effectiveOC.png'
tags: OC iOS 
---

* [熟悉Objective-C](#one)
    * [第1条:了解Objective-C语言的起源](#1)
    * [第2条:在类的头文件中尽量少引用其他头文件](#2)
    * [第3条:多用字面量语法,少用与之等价的方法](#3)
    * [第4条:多用类型常量,少用#define预处理指令](#4)
    * [第5条:用枚举表示状态,选项,状态码](#5)
* [对象,消息,运行期](#two)
    * [第6条:理解"属性"这一概率](#6)
    * [第7条:在对象内部尽量直接访问实例变量](#7)
    * [第8条:理解"对象等同性"这一概念](#8)
    * [第9条:以"类族模式"隐藏实现细节](#9)
    * [第10条:在既有类中使用关联对象存放自定义数据](#10)
    * [第11条:理解objc_msgSend的作用](#11)
    * [第12条:理解消息转发机制](#12)
    * [第13条:用"方法调配技术"调试"黑盒方法"](#13)
    * [第14条:理解"类对象"的用意](#14)
* [接口与API设计](#three)
* [协议与分类](#four)
* [内存管理](#five)
* [block与GCD](#six)
* [系统框架](#seven)

<h3 id="one">熟悉Objective-C</h3>

<h4 id="1">第1条:了解Objective-C语言的起源</h4>

* Objective-C使用动态绑定的消息结构,在运行时才会检查对象类型;
* 理解C语言的核心概念,掌握内存模型与指针.

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

<h4 id="6">第6条:理解"属性"这一概率</h4>

如果使用了属性,编译器会自动编写访问这些属性所需的方法,此过程叫"自动合成".这个过程由编译器在编译期执行,还要自动向类中添加适当类型的实例变量,并且在属性名前加下划线,以此作为实例变量的名字.

* 使用@synthesize可指定所生成实例变量的名字,不使用@synthesize也能生成实例变量和存取方法,实例变量的名字默认为属性名前加下划线;
* 使用@dynamic会告诉编译器不要自动创建实现属性所用的实例变量,也不要为其创建存取方法.

<h5>属性特质</h5>

* 原子性
    * atomic会使用同步锁来确保由编译器合成的方法具有原子性(并发编程中,操作的整体性,就是说系统其他部分无法观察到其中间步骤所生成的临时结果,只能看到操作前后的结果),nonatomic不会有这种特性.若自己定义存取方法,应与属性特质相符.
* 读/写权限

    * 默认为readwrite,同时拥有存取方法;
    * readonly只会自动生成获取方法.
* 内存管理语义
    * assign:只会执行针对"纯量类型"的简单赋值操作;
    * strong:针对对象,跟retain一样,设置方法会retain新值,release旧值;
    * weak:不会引用对象,对象销毁时值会为nil;
    * unsafe_unretained:对象销毁时,属性值不会清空为nil;
    * copy:与strong类似,不会retain新值,会copy新值,同样使引用计数+1;
* 方法名
    * getter=<name> 指定"获取方法"的方法名;
    * setter=<name> 指定"设置方法"的方法名;
    
    注意在其他方法里设置属性值也要遵守属性定义中的语义.比如在初始化方法中设置copy修饰的字符串属性的值.

<h4 id="7">第7条:在对象内部尽量直接访问实例变量</h4>

* 在对象内部读取数据时,应该直接通过实例变量来读,而写入数据时,则应通过属性来写;
* 在初始化方法及dealloc方法中,总是应该直接通过实例变量来读写数据;
* 使用惰性初始化技术配置数据,需要通过属性来读取数据;

<h4 id="8">第8条:理解"对象等同性"这一概念</h4>

若想检测对象的等同性,请提供isEqual:与hash方法.
isEqual:的执行过程:
```
- (BOOL)isEqual:(id)object {
    if(self == object) return YES;
    if([self class] != [object class]) return NO;
    Person *otherPerson =(Person *)object;
    if(![_name isEqualToString:otherPerson.name]) return NO;
    if(_age != otherPerson.age) return NO;
    return YES;
}
```
> 一般通过关键属性来检测,也不要盲目地逐条检测每个属性,应该依照具体需求制定方案,还要注意继承体系中判断等同性的情况.

hash值通过关键属性的按位异或来计算:
```
- (NSUInteger)hash {
    NSUInteger nameHash = [_name hash];
    NSUInteger ageHash = _age;
    return nameHash ^ ageHash;
}
```
>hash方法只在对象被添加至NSSet和设置为NSDictionary的key时会调用.

>还要一种情况一定要注意,就是在容器中放入某个对象,就不应改变其hash码.解决这个问题,要保证放入容器后不再改变对象内容.

<h4 id="9">第9条:以"类族模式"隐藏实现细节</h4>
类族模式可以把实现细节隐藏在一套简单的公共接口后面.系统框架经常使用类族.从类族的公共抽象基类中继承子类时要当心.

<h4 id="10">第10条:在既有类中使用关联对象存放自定义数据</h4>
在分类中创建的属性是没有生成成员变量的,因此无法使用,可通过关联对象的方式来构建成员变量.如kvo的封装.

<h4 id="11">第11条:理解objc_msgSend的作用</h4>
某函数的最后一项操作是调用另外一个函数,就可以运用"尾调用优化"技术.编译器会生成跳转至另一函数所需的指令码,而且不会向调用堆栈中推入新的"栈帧".objc_msgSend函数会利用这一技术,令"跳至方法实现"变得更简单.

<h4 id="12">第12条:理解消息转发机制</h4>
消息转发分为两大阶段:

* "动态方法解析":
<h4 id="13">第13条:用"方法调配技术"调试"黑盒方法"</h4>
<h4 id="14">理解"类对象"的用意</h4>

<h3 id="three">接口与API设计</h3>
<h3 id="four">协议与分类</h3>
<h3 id="five">内存管理</h3>
<h3 id="six">block与GCD</h3>
<h3 id="seven">系统框架</h3>