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
    * [第15条:用前缀避免命名空间冲突](#15)
    * [第16条:提供"全能初始化方法"](#16)
    * [第17条:实现description方法](#17)
    * [第18条:尽量使用不可变对象](#18)
    * [第19条:使用清晰而协调的命名方式](#19)
    * [第20条:为私有方法名加前缀](#20)
    * [第21条:理解Objective-C错误模型](#21)
    * [第22条:理解NSCopying协议](#22)
* [协议与分类](#four)
    * [第23条:通过委托与数据源协议进行对象间通信](#23)
    * [第24条:将类的实现代码分散到便于管理的数个分类之中](#24)
    * [第25条:总是为第三方类的分类名称加前缀](#25)
    * [第26条:勿在分类中声明属性](#26)
    * [第27条:使用"class-continuation分类"隐藏实现细节](#27)
    * [第28条:通过协议提供匿名对象](#28)
* [内存管理](#five)
    * [第29条:理解引用计数](#29)
    * [第30条:以ARC简化引用计数](#30)
    * [第31条:在dealloc方法中只释放引用并解除监听](#31)
    * [第32条:编写"异常安全代码"时留意内存管理问题](#32)
    * [第33条:以弱引用避免保留环](#33)
    * [第34条:以"自动释放池"降低内存峰值](#34)
    * [第35条:用"僵尸对象"调试内存管理问题](#35)
    * [第36条:不要使用retainCount](#36)
* [block与GCD](#six)
* [系统框架](#seven)
    * [第47条:熟悉系统框架](#47)
    * [第48条:多用块枚举,少用for循环](#48)
    * [第49条:对自定义其内存管理语义的collection使用无缝桥接](#49)
    * [第50条:构建缓存时选用NSCache而非NSDictionary](#50)
    * [第51条:精简initialize与load的实现代码](#51)
    * [第52条:别忘了NSTimer会保留其目标对象](#52)

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
给对象发送消息,会先在本类中搜索方法的实现,有就直接调用若没有则去父类查找直到NSObject,如果NSObject没有则进去消息转发:

* "动态方法解析":看能否动态添加方法,以处理这个"未知的选择器",其返回值表示这个类能否新增一个实例方法用以处理此选择器.使用这种办法的前提是相关代码已经写好,只等着运行的时候动态插在类里面.
    ```
    + (BOOL)resolveInstanceMethod:(SEL)sel {
        return [super resolveInstanceMethod:sel];
    }
    + (BOOL)resolveClassMethod:(SEL)sel {
        return [super resolveClassMethod:sel];
    }
    ```

* 备援接收者:若当前接收者能找到备援对象则将其返回,若找不到就返回nil;
    ```
    - (id)forwardingTargetForSelector:(SEL)aSelector {
        return nil;
    }
    ```
* 完整的消息转发:实现此方法时,若发现某调用操作不应由本类处理,则需调用超类的同名方法.这样的话,继承体系中的每个类都有机会处理此请求,直至NSObject.NSObject会调用"doesNotRecognizeSelector:"
    ```
    -(void)forwardInvocation:(NSInvocation*)anInvocation {
    }
    - (NSMethodSignature*)methodSignatureForSelector:(SEL)aSelector {
    }
    ```
实际运用:[利用NSProxy实现消息转发-模块化的网络接口层设计](https://blog.csdn.net/xiaochong2154/article/details/44886973)

<h4 id="13">第13条:用"方法调配技术"调试"黑盒方法"</h4>

```
Class class = [self class];
    // 原方法名和替换方法名
    SEL originalSelector = @selector(viewDidAppear:);
    SEL swizzledSelector = @selector(swizzle_viewDidAppear:);
    // 原方法结构体和替换方法结构体
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    // 如果当前类没有原方法的实现IMP，先调用class_addMethod来给原方法添加默认的方法实现IMP
    BOOL didAddMethod = class_addMethod(class,originalSelector,method_getImplementation(swizzledMethod),method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {// 添加方法实现IMP成功后，修改替换方法结构体内的方法实现IMP和方法类型编码TypeEncoding
        class_replaceMethod(class,swizzledSelector,method_getImplementation(originalMethod),method_getTypeEncoding(originalMethod));
    } else { // 添加失败，调用交互两个方法的实现
        method_exchangeImplementations(originalMethod, swizzledMethod);
    } 
```
> const char *types:"v@:"意思就是这已是一个void类型的方法，没有参数传入;"i@:"就是说这是一个int类型的方法，没有参数传入。"i@:@"就是说这是一个int类型的方法，又一个参数传入。

<h4 id="14">理解"类对象"的用意</h4>

```
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};
typedef struct objc_object *id;

typedef struct objc_class *Class;
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;

```

* 每个对象结构体的首个成员是Class类型的变量,称为"isa"指针.该变量定义了对象所属类;
* Class对象结构体存放类的"元数据"(metadata),包括类的成员变量,实例方法,协议列表,超类等等.
* Class对象结构体的首个变量也是isa指针,说明它也是OC对象.对象所属类型(也就是isa指针指向类型),叫做"元类"(metaclass)."类方法"定义与此处.
* 每个类仅有一个Class对象,每个Class对象仅有一个"元类"对象.

假设有个SomeClass的子类从NSObject中继承,继承体系如图所示:
![objc_class](https://raw.githubusercontent.com/w-qihang/w-qihang.github.io/master/_posts/imgs/objc_class.png)

<h5>在继承体系中查询类型信息</h5>

* "isMemberOfClass:"能够判断对象是否是某个特定类的实例;
* "isKindOfClass:"则能判断出对象是否为某类或其派生类的实例.

还有一种可以精确判断对象是否为某类实例的办法:

```
if([obj class] == [Person class]) {
        //'obj' is an instance of Person
    }
```
尽量可以这样做,但也要尽量使用类型信息查询方法来确定对象类型.因为某些对象可能实现了消息转发功能.
> NSProxy代理对象虽遵循了NSObject协议,但在调用isKindOfClass:时,并未实现此方法,要把消息转发给"接受代理的对象".

<h3 id="three">接口与API设计</h3>

<h4 id="15">第15条:用前缀避免命名空间冲突</h4>
选择与公司应用程序或二者皆有关联之名称作为类名的前缀,并在所有代码中均使用这一前缀,若自己开发的程序库中用到了第三方库,则应为其中的名称加上前缀.

<h4 id="16">第16条:提供"全能初始化方法"</h4>

* 在类中提供一个全能初始化方法,并于文档中指明.其他初始化方法均应调用此方法.
* 若全能初始化方法与超类不同,则需覆写超类中对应方法.
* 如果超类的初始化方法不适用于子类,那么应该覆写这个超类方法,并在其中抛出异常.

<h4 id="17">第17条:实现description方法"</h4>

* 在自定义的description方法中,把待打印的信息放在字典里,然后将字典对象的description方法所输出的内容包含在字符串里返回("object=%@"会调用到description).
* debugDescription方法是开发者在调试器中以控制器命令打印对象时才调用的.(po命令)

<h4 id="18">第18条:尽量使用不可变对象"</h4>

* 尽量创建不可变的对象.
* 若某属性仅可用于对象内部修改,则在扩展中将其由readonly扩展为readwrite.
* 不要把可变的collection作为属性公开,而应提供相关方法,以此修改对象中的可变collection.

<h4 id="19">第19条:使用清晰而协调的命名方式"</h4>
    给方法起名时的第一要务就是确保其风格与你自己的代码或所要集成的框架相符.

<h4 id="20">第20条:为私有方法名加前缀"</h4>

* 给私有方法加上前缀,可以很容易地将其同公共方法分开.
* 不要单用一个下划线做私有方法前缀,因为这种做法是预留给苹果公司用的.

<h4 id="21">第21条:理解Objective-C错误模型"</h4>

* 只有发生了可使整个应用程序崩溃的严重错误时,才应使用异常.
* 在错误不那么严重的情况下,可以指派"委托方法"来处理错误,也可以把错误信息放在NSError对象里,经由"输出参数"返回给调用者.

<h4 id="22">第22条:理解NSCopying协议"</h4>

* 若要令自己所写的对象具有拷贝功能,则需实现NSCopying协议.copy方法由NSObject实现,会调用"copyWithZone:"方法.,所以我们不要覆写copy方法.
* 如果自定义的对象分为可变与不可变版本,那要同时实现NSCopying与NSMutableCopying协议.
* 复制对象时需决定采用浅拷贝还是深拷贝,一般情况下应该尽量执行浅拷贝.
* 如果你写的对象需要深拷贝,那么可考虑新增一个专门执行深拷贝的方法.

<h3 id="four">协议与分类</h3>

<h4 id="23">第23条:通过委托与数据源协议进行对象间通信"</h4>

* 将委托对象应该支持的接口定义成协议,在协议中把可能需要处理的事件定义成方法.
* 将数据源协议与委托协议分离,能使接口更加清晰."数据源"与"受委托者"可以是两个不同的对象.
* 可实现含有位段的结构体,将委托对象是否能响应相关协议方法这一信息缓存至其中.

<h4 id="24">第24条:将类的实现代码分散到便于管理的数个分类之中"</h4>

* 使用分类机制把类的实现代码划分成易于管理的小块.
* 将应该视为"私有"的方法归入名叫Private的分类中,以隐藏实现细节.

<h4 id="25">第25条:总是为第三方类的分类名称加前缀"</h4>
向第三方类中添加分类时,给其名称和其中的方法加上你专用的前缀.

<h4 id="26">第26条:勿在分类中声明属性"</h4>
把封装数据所用的全部属性都定义在主接口里.在分类中可以定义存取方法,但尽量不要定义属性.

<h4 id="27">第27条:使用"class-continuation分类"隐藏实现细节"</h4>

* 通过类扩展向类中新增实例变量以实现私有.
* 如果某属性在主接口中声明为"只读",而类的内部又要用设置方法修改此属性,那么就在类扩展中将其扩展为"可读写".
* 把私有方法的原型声明在类扩展里面.
* 若想使类所遵循的协议不为人所知,在类扩展中声明.

<h4 id="28">第28条:通过协议提供匿名对象"</h4>

* 协议可在某种程度上提供匿名类型.具体的对象类型可以淡化成遵从某协议的id类型,协议里规定了对象所应实现的方法.
* 使用匿名对象来隐藏类型名称.

<h3 id="five">内存管理</h3>

<h4 id="29">第29条:理解引用计数"</h4>

* 引用计数机制通过可以递增递减的计数器来管理内存.计数将为0时,对象就销毁了.
* 保留和释放操作分别会递增及递减保留计数.

<h4 id="30">第30条:以ARC简化引用计数"</h4>
    ARC只负责OC对象的内存.CoreFoundation对象不归ARC管理,开发者必须适时调用CFRetain/CFRelease.

<h4 id="31">第31条:在dealloc方法中只释放引用并解除监听"</h4>

* 在dealloc方法里,应该做的事情就是释放指向其他对象的引用,并取消原来订阅的KVO或者NSNotification等通知,不做其他事情.
* 如果对象持有文件描述符等资源,那么应该专门编写一个方法来释放此种资源.这样的类要和其使用者约定:用完资源后必须调用close方法.
* 执行异步任务的方法不应在dealloc里调用;只能在正常状态下执行的那些方法也不应在dealloc里调用,因为此时对象已处于正在回收的状态了.

<h4 id="32">第32条:编写"异常安全代码"时留意内存管理问题"</h4>

* 捕获异常时,一定要注意将try块内所创立的对象清理干净.
* 在默认情况下,ARC不生成安全处理异常所需的清理代码.开启编译器标志后(-fobjc-arc-exceptions),可生成这种代码,不过会导致应用程序变大,而且降低运行效率.

<h4 id="33">第33条:以弱引用避免保留环"</h4>
    在具备自动清空功能的弱引用上,可以随意读取其数据,因为这种引用不会指向已经回收过的对象.

<h4 id="34">第34条:以"自动释放池"降低内存峰值"</h4>
    合理运用自动释放池,可降低应用程序的内存峰值.

<h4 id="35">第35条:用"僵尸对象"调试内存管理问题"</h4>

* 系统在回收对象时,可以不将其真的回收,而是把它转化为僵尸对象.通过环境变量NSZombieEnabled可开启此功能.
* 系统会修改对象的isa指针,令其指向特殊的僵尸类,从而使该对象变为僵尸对象.僵尸类能够响应所有的选择子,响应方式为:打印一条包含消息内容及其接受者的消息,然后终止应用程序.

<h4 id="36">第36条:不要使用retainCount</h4>
    对象的引用计数看似有用,实则不然,因为任何给定时间点上的"绝对保留计数"都无法反映对象生命期的全貌.引入ARC之后,retainCount就正式废弃了.

<h3 id="six">block与GCD</h3>

<h3 id="seven">系统框架</h3>

<h4 id="47">第47条:熟悉系统框架"</h4>

* 许多系统框架都可以直接使用.其中最重要的是Foundation与CoreFoundation,这两个框架提供了构建应用程序所需的许多核心功能.
* 用纯c写成的框架与用Objective-C写成的一样重要.应该掌握c语言的核心概念.

<h4 id="48">第48条:多用块枚举,少用for循环</h4>

* 遍历collection有四种方式.最基本的是for循环,其次是NSEnumerator遍历法及快速遍历法,最新,最先进的方式则是"块枚举法".
* "块枚举法"本身就能通过GCD来并发执行遍历操作,无须另行编写代码.而采用其他遍历方式则无法轻易实现这一点.
* 若提前知道待遍历的collection含有何种对象,则应修改块签名,指出对象的具体类型.

<h4 id="49">第49条:对自定义其内存管理语义的collection使用无缝桥接</h4>

用__bridge转换为CF对象,ARC仍然具备这个OC对象的所有权.若是用__bridge_retained来实现,意味着ARC将交出对象所有权,用完数组后要加上CFRelease以释放其内存.想把CFArrayRef转换为NSArray*,并且想令ARC获得对象所有权,那么就采用__bridge_transfer来实现.下面是对数组简单的一段无缝桥接代码
```
NSArray *anNSArray = @[@1, @2, @3, @4, @5];
CFArrayRef aCFArray = (__bridge CFArrayRef)anNSArray;
NSLog(@"size of array = %li",CFArrayGetCount(aCFArray));
```
在CoreFoundation层面创建collection时,可以指定许多回调函数,这些函数表示此collection应如何处理其元素.然后可运用无缝桥接技术将其转换成具备特殊内存管理语义的Objective-C collection.
```
static const void* retainCallback(CFAllocatorRef allocator, const void *value) {
    return CFRetain(value);
}
static void releaseCallback(CFAllocatorRef allocator, const void *value) {
    CFRelease(value);
}
static NSMutableDictionary* createDic() {
    CFDictionaryKeyCallBacks keyCallbacks = {
        0,
        retainCallback,
        releaseCallback,
        NULL,
        CFEqual,
        CFHash
    };
    CFDictionaryValueCallBacks valueCallbacks = {
        0,
        retainCallback,
        releaseCallback,
        NULL,
        CFEqual
    };
    CFMutableDictionaryRef aCFDictionary = CFDictionaryCreateMutable(NULL, 0, &keyCallbacks, &valueCallbacks);
    NSMutableDictionary *anNSDictionary = (__bridge_transfer NSMutableDictionary*)aCFDictionary;
    return anNSDictionary;
}
```

<h4 id="50">第50条:构建缓存时选用NSCache而非NSDictionary</h4>

* 实现缓存时应选用NSCache而非NSDictionary对象.因为NSCache可以提供优雅的自动删减功能,而且是"线程安全的",此外它还会删减"最久未使用对象",它与字典不同,并不会拷贝键.
* 可以给NSCache对象设置上限,用以限制缓存中的对象总个数及"总开销",而这些尺度则定义了缓存删减其中对象的时机.但是绝对不要把这些尺度当成可靠的"硬限制",它们仅对NSCache起指导作用.应该在很快能计算出"开销值"的情况下,才考虑限制总开销.
* 将NSPurgeableData与NSCache搭配使用,可实现自动清除数据的功能,当NSPurgeableData对象所占内存为系统所丢弃时,该对象自身也会从缓存中移除.
* 只有那种"重新获取起来很费事的"数据,才值得放入缓存,比如需要从网络获取或从磁盘读取的数据.

<h4 id="51">第51条:精简initialize与load的实现代码</h4>

* 在加载阶段,如果类实现了load方法,那么系统就会调用它.分类里也可以定义此方法,类的load方法要比分类中的先调用.与其他方法不同,load方法不参与覆写机制.
* 首次使用某个类之前,系统会向其发送initialize消息.由于此方法遵从普通的覆写规则,所以通常应该在里面判断当前要初始化的是哪个类.
* load与initialize方法都应该实现的精简一些,这有助于保持应用程序的响应能力,也能减少引入"依赖环"的几率.
* 无法在编译期设定的全局常量,可以放在initialize方法里初始化.

<h4 id="52">第52条:别忘了NSTimer会保留其目标对象</h4>

* NSTimer对象会保留其目标,直到定时器本身失效.调用invalidate可令定时器失效,一次性的定时器在触发完任务之后也会失效.
* 反复执行任务的计时器很容易引入保留环,如果计时器的目标对象又保留了计时器本身,那肯定会导致保留环.
打破保留环的方法:

* 手动调用invalidate令定时器失效.
* 用block来打破保留环.block里面弱引用目标对象self.
    ```
    //ios10已提供此方法
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block
    ```
* NSProxy对象作为NSTimer的目标对象,弱引用self对象,并将消息转发给self对象.
```
@interface LSYProxy : NSProxy
@property (nonatomic,weak) id obj;
@end

LSYProxy *proxy = [LSYProxy alloc];
proxy.obj = self;
self.timer = [NSTimer scheduledTimerWithTimeInterval:5.0 target:proxy selector:@selector(timerEvent) userInfo:nil repeats:YES];

@implementation LSYProxy
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSMethodSignature *sig;
    sig = [self.obj methodSignatureForSelector:aSelector];
    return sig;

}
- (void)forwardInvocation:(NSInvocation *)invocation {
    [invocation invokeWithTarget:self.obj];
}
@end
```