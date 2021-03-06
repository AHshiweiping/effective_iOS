#前言

Segment Fault：http://segmentfault.com/a/1190000003694112

（首发在 Segment Fault）

这是一篇读书笔记，快速记录各种高效率编程的技巧和方法。这些方法是为了提升编码质量和效率，高质量代码利于后期的维护和更新，毕竟不能一份代码到永远。

由于是记录形式，当然不能把整篇内容都写下来，只记录关键性的内容，长期更新。

#正文

`Objective-C`使用了消息机制代替调用方法。

区别：使用消息结构的语言，其运行时缩影执行的代码由运行环境来决定。而使用函数调用的语言，则又编译器决定。

###头文件中少引用其他文件

在头文件中使用`@Class`代替直接引用其他头文件

###多使用字面量语法

```objective-c
    NSNumber *intNumber = @1;
    NSNumber *floatNumber = @2.5f;
    NSNumber *doubleNumber = @3.1415926;
    NSNumber *boolNumber = @YES;
    NSNumber *charNumber = @'a';
    
    int a = 3;
    float b = 2.1;
    NSNumber *c = @(a*b);
    
    NSArray *animals = @[@"cat",@"dog",@"monkey"];
    
    NSString *dog = animals[1];

    NSDictionary *dataDict = @{ @"firstName" : @"aa",
                                @"lastName" : @"bb",
                                @"age" : @20 };
    
    NSString *lastName = dataDict[@"lastName"];
    
    NSMutableArray *mutableArray = animals.mutableCopy;
```

###多用类型常量，少用`#define`预处理

如果只在本类使用的常量，使用`static const`关键字来定义常量。

如果多个类都需使用到某一常量，则需将常量定义成公开的，具体方式是在类的声明文件中使用`extern const`关键字声明常量，在类的实现文件中使用`const`关键字定义常量，这样任何类只要导入了声明常量的头文件就可以直接使用定义好的常量了。

在`.h`文件中声明

```
extern NSString *const XFExternalConst;
```
在`.m文件中`描述

```
NSString *const XFExternalConst = @"ko";
```
为避免冲突，一般都用类名做前缀。

###用枚举表示状态、选项、状态码

枚举只是一种常量命名方式，某个对象所经历的各种状态可以定义为一个枚举集。

编译器会为枚举分配一个独有的编号，从0开始每个递增加1.实现枚举所用的数据类型取决于编译器，不过其二进制位的个数必须能完全表示枚举编号才行。

```
enum ConnectionState {
    ConnectionStateDisconnected,
    ConnectionStateConnecting,
    ConnectionStateConnected
};

typedef enum ConnectionState ConnectingState;

```

还可以不使用编译器所分配的编号，手工指定某个枚举成员所对应的值。

还有一种情况应该使用枚举类型，那就是定义选项的时候。若这些选项可以彼此组合，则更应该如此。只要枚举定义的对，各选项之间就可以通过“按位或操作符”来组合。


凡是需要以按位或操作来组合的枚举都应该用`NS_OPTIONS`宏，如果没有组合需求，就用`NS_ENUM`宏。
```
typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {
    UIViewAnimationTransitionNone,
    UIViewAnimationTransitionFlipFromLeft,
    UIViewAnimationTransitionFlipFromRight,
    UIViewAnimationTransitionCurlUp,
    UIViewAnimationTransitionCurlDown,
};

typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```
枚举在`switch`语句里面的时候，不需要加`default`分支。

###属性的概念

基本方法就不描述了。

`@dynamic`关键字，表示不要自动创建实现属性所有的实例变量，也不要为其创建存取方法。即使编译器没有发现定义存取方法，也不会报错，它相信这些方法能在运行期找到。

属性的四种特质

- 原子性

默认情况下，编译器合成的方法会锁定机制保持`atomic`。如果使用`nonatomic`，则不使用同步锁。

- 读写权限

`readwrite`的属性具有`getter`和`setter`方法

`readonly`的属性仅具有`getter`方法

- 内存管理语义

`assign`只针对“纯量类型”，比如`CGFloat`或者`NSInteger`

`strong`表示该属性定义了一种`拥有关系`。为这种属性设置新值时，设置方法会先保留新值，并释放旧值，然后将新值设置上去

`weak`表示该属性定义另一种`非拥有关系`。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。和`assign`类似，然而在属性所指的对象遭到摧毁时，属性值也会清空

`unsafe_unretained`这个和`assign`相同，但是它适用于`对象类型`，该特质表达一种`非拥有关系`，当目标对象遭到摧毁时，属性值不会自动清空，是不安全的

`copy`表达的所属关系和`strong`类型。然后设置方法并不保留新值，而是将其拷贝。当属性类型为`NSString *`时，经常用此特质来保护其封装性，因为传递给设置方法的新值可能指向一个`NSMutableString`类的实例。如果不是拷贝的花，那么设置完属性以后，字符串的值可能会在对象不知情的情况下遭人更改。所以这个时候需要拷贝一份不可变的字符串。

- 方法名

`getter=<name>` 指定`getter`的方法名。如果属性是`Boolean`型，在方法名加上`is`前缀，就可以用这个方法来指定。

`setter=<name>` 指定`setter`的方法名。这个不常见。

###在对象内部尽量直接访问实例变量

懒加载是重写`getter`方法

###理解`对象等同性`的概念

按照`==`操作符比较出来的结果未必是我们想要的，因为该操作符比较出来的是两个指针本身，而不是指针所指的对象。应该是用`NSObject`协议中声明的`isEqual`方法来判断两个对象的等同性。来办来说两个类型不同的对象总是不相等的。

    NSString *oneStr = @"aaa 21";
    NSString *twoStr = [NSString stringWithFormat:@"aaa %d",21];
    BOOL equalA = (oneStr == twoStr);//NO
    BOOL equalB = [oneStr isEqual:twoStr];//YES
    BOOL equalC = [oneStr isEqualToString:twoStr];//YES

两个用于判断等同性的关键方法

    - (BOOL)isEqual:(id)object;
    @property (readonly) NSUInteger hash;

默认实现是：当且仅当其指针值完全相等时，这两个对象才相等。

**几个要点**

- 若想监测对象的等同性，提供`isEqual:`与 hash 方法
- 相同的对象必须具有相同的哈希码，但是两个哈希码相同的对象却未必相同
- 不要盲目逐个检测每条属性，而是应该依照具体需求来制定监测方案
- 编写 hash 方法时，应该是用计算速度快而且碰撞低的算法

###以“类族模式”隐藏实现细节

核心套路就是类似`UIButton`，创建的时候传入一个枚举值，根据枚举值来创建子类。（这里的笔记是我看懂以后写的，不知道的朋友先搜索一下`工厂模式`，其实就是那个意思）

###在既有类中使用关联对象存放自定义数据

有时候需要在对象中存放相关信息，这时候我们通常会从对象所属的类中继承一个子类，然后改用这个子类对象。然而并非所有情况下都能这么做，有时候类的实例可能由某种机制创建。`Objective-C`有一种强大机制叫`关联对象`。

这种机制要小心使用，因为会使代码失控。

###理解`objc_msgSend`的作用

原型

```
void objc_msgSend(id self, SEL cmd, ...)
```

一个例子

```
id returnValue = [someObject messageName:parameter];
```
编译器会把它转换为以下函数

```
id returnValue = objc_msgSend(someObject,@selector(messageName:),parameter);
```

为了完成调用方法，该方法需要在接受者所属的类中搜寻其`方法列表`，如果能找到，就跳转过去。如果找不到，就沿着继承体系向上继续查找，等找到合适的再跳转。如果最终还是找不到，就执行`消息转发`的操作。

一些**边界情况**，则交由另一些函数处理

- `objc_msgSend_stret` 如果待发送的消息要返回结构体，可交此函数处理。
- `objc_msgSend_fpret` 如果消息返回的是浮点数，可交由此函数处理。
- `ojbc_msgSendSuper` 如果要给超类发送消息，例如`[super message:parameter]`，那么就就交由此函数处理。

###理解消息转发机制

消息转发分为两大阶段。

第一阶段先问接受者，所属的类，看其是否能动态添加方法，以处理当前这个`unknown selector`，这称为`dynamic method resolution`。

第二阶段涉及`full forwarding mechanism`。如果运行期系统已经把第一阶段执行完了，那么接受者自己就无法再以动态新增方法的手段来响应包含该`selector`的消息了。此时，运行期系统会请求接受者以其他手段来处理与消息相关的方法调用。然后又分两部。

首先，让接受者看看有没有其他对象能处理这条消息。如果有，就转发给那个对象。

如果没有，就会启动完整的消息转发机制，运行期系统会把消息有关的全部细节封装到`NSInvocation`对象中，再给接受者最后一次机会，让它设法解决当前还未处理的这条消息。

**动态方法解析**

对象收到无法解读的消息后，先调用

```
+ (BOOL)resolveInstanceMethod:(SEL)sel 
```

该方法的参数就是那个未知的`selector`，返回`Boolean`类型，表示这个类是否能新增一个实例方法用来处理这个`selector`。在继续走下去之前，这有个机会新增一个处理的方法。

如果尚未实现的不是实例方法而是类方法，则调用

```
+ (BOOL)resolveClassMethod:(SEL)sel
```

使用他们的前提是，相关方法的实现代码已经写好，只等着运行的时候动态插入在类里面。

这个常常用来实现`@dynamic`属性。

**后备接收者**

当前接收者还有第二次机会处理，能不能把消息转发给其他接收者

```
- (id)forwardingTargetForSelector:(SEL)aSelector
```
找得到就返回对象，找不到就返回`nil`。

**完整的消息转发**

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```
先创建`NSInvocation`对象，把尚未处理的那条消息有关的全部细节都封在其中。此对象包含`selector`，`target`以及参数。

继承体系中的每个类都有机会处理此调用请求，直到`NSObject`。如果还没有找到，那么该方法还会继续调用`doesNotRecognizeSelector:`抛出异常，此异常表示最终未能处理。

这个机制属于底层机制，可以动态注入方法，甚至之前的可以动态注入属性，云后端服务商可以说基本就靠这个套路，通过KVC的样子往类里面添加属性。

###用方法调配技术调试黑盒方法

黑科技。

IMP指针，改方法实现，替换系统方法，可以多添加日志打印。

###类对象

OC 是一门极其动态的语言。

每个 OC 对象实例都是指向某块内存数据的指针。

```
typedef struct objc_object {
    Class isa;
} *id;
```
每个对象结构体的首个成员是`Class`类的变量。该变量定义了对象所属的类，通常称为`is a`指针。

```
typedef struct objc_class *Class;
struct objc_class {
    Class isa;
    Class super_class;
    const char *name;
    long version;
    long info;
    long instance_size;
    struct objc_ivar_list *ivars;
    struct objc_method_list **methodLists;
    struct objc_cache *cache;
    struct objc_protocol_list *protocols;
};
```
此结构体存放类的`元数据`，例如类的实例实现了几个方法，具备多少个实例变量等信息。
首个变量是`isa`指针，说明`Class`本身也是 OC 对象。

`super_class`定义了本类的超类。类对象所属的类型是另一个类，叫做`超类`。

每个类仅有一个`类对象`，而每个`类对象`仅有一个与之相关的`元类`。

`class`方法所返回的类表示发起代理的对象，而非接受代理的对象。

###用前缀避免命名空间冲突

开发者可能会忽视另外一个容易引发命名冲突的地方，那就是类的实现文件中所用的纯 C 函数及全局变量。 

###提供全能初始化方法

所有对象均要初始化。

提供一个全能初始化方法，其他的几种初始化方法调用它。

如果全能初始化方法与超类不同，则需覆写超类中的对应方法。

###实现`description`方法

重写`- (NSString *)description`

控制台`- (NSString *)debugDescription`

###尽量使用不可变对象

尽量把对外公布出来的属性设为只读，只在必要时候对外公布。

有时候想修改封装在对象内部的数据，但是却不想让外人所改动。这种情况需要将`readonly`在`.m`文件中重新生成`readwrite`。但是为了避免产生意外，需要在必要时通过`dispatch queue`来实现。

不要把可变的内容作为属性公开，而是提供相关方法，以此修改对象中的可变内容。

###使用清洗而协调的命名方式

驼峰命名法

方法与变量以`小写字母`开头

类名以`大写字母`开头

不要使用`str`这种简称，而用`string`这样的全称

`Boolean`属性应该加`is`前缀，如果某方法返回非属性的`Boolean`值，应该根据功能选用`has`或者`is`当前缀

**类与协议的命名**

为类与协议的名称加上前缀，以避免命名空间的冲突

委托一般使用委托的发起方名称后面跟一个`Delegate`


###为私有方法名加前缀

一般可以使用`p_`作为前缀，表示私有方法

不要用一个单独的下划线作为私有方法的前缀

###理解`Objective-C`错误模型

异常`NSException`应该用于极其严重的错误，比如编写了某个抽象基类，它的正确用法是先从重继承一个子类，然后再使用这个子类。在这种情况下，如果有人直接使用了这个抽象基类，那么可以考虑抛出异常。

`NSError`的用法很灵活，封装了三条信息

- `Error domain` 错误范围，类型为字符串
错误发生的范围，通常用一个特有的全局变量来定义。

- `Error code` 错误码，类型为证书
独有的错误代码。这种错误通常采用`enum`来定义，比如 HTTP 请求返回的状态码。

- `User info` 用户信息，类型为字典
有关此错误的额外信息，其中或许包含一段*本地化描述*，或许还包含导致该错误发生的另外一个错误，经由此种信息，可将相关错误传承一条`chain of errors`。


###理解`NSCopying`协议

使用对象时经常需要拷贝它。如果想令自己的类支持拷贝操作

```
- (id)copyWithZone:(NSZone *)zone;
```
为什么会出现`NSZone`，以前开发的时候，会把内存分成不同的`zone`，而对象会创建在某个区里面。现在不用了，每个程序只有一个`default zone`

另外一个`NSMutableCopying`协议，返回可变的副本

```
- (id)mutableCopyWithZone:(NSZone *)zone;
```

**深拷贝**
在拷贝对象自身时，将底层数据也一并复制过去

**浅拷贝**
`Foundation`框架中所有的容器类默认情况下执行浅拷贝，只拷贝对象本身，不复制数据
因为不是所有对象都能拷贝，而且调用者也未必需要都一一拷贝。


###通过委托与数据源协议进行对象间通信

委托属性要定义成`weak`，因为两者之间必须为`非拥有关系`

```
- (BOOL)respondsToSelector:(SEL)aSelector;
```

也可以用协议定义一套接口，令某类从该接口获取所需的数据。委托模式的这种用法是向类提供数据，所以成为`dataSource`。在这种模式中，信息从数据源流向类。而在常规的代理模式中，信息则从类流向受委托者。


###将类的实现代码分散到便于管理的数个分类之中

把一个类中的几个不同模块方法写到别的文件中，合理使用`category`。

###不要在分类中声明属性

除了`extension`外，其他的分类都无法向类中新增实例变量

声明为`@dynamic`，然后动态添加


###使用`extension`隐藏实现细节


###通过协议提供匿名对象

使用匿名对象来隐藏类型名称

###理解引用计数

`retain` 增计数
`release` 减计数
`autorelease` 待稍后清理`autorelease pool`时，再减少计数

对象创建出来时，其保留计数至少为1

**自动释放池**

**循环引用**

###以ARC简化引用计数

若方法名以下列词语开头，则返回的对象归调用者所有
- alloc
- new
- copy
- mutableCopy

在应用程序中，可用下列修饰符来改变局部变量与实例变量的语义

`__strong` 默认语义，保留这个值

`__unsafe_unretained` 不保留这个值，这么做可能不安全，因为等到再次使用变量时，其对象可能已经回收了

`__weak` 不保留这个值，但是变量可以安全使用，因为如果系统把这个对象回收了，那么变量也会自动清空

`__autoreleasing` 把对象*按引用传递*给方法时，使用这个特殊的修饰符，此值在方法返回时自动释放

比如，想令实例变量的语义与不使用 ARC 时相同，可以使用`__weak`或`__unsafe_unretained`修饰符

block 块会自动保留其所捕获的全部对象，而如果这其中有某个对象又保留了块本身，那么就可能导致循环引用，可以用`__weak`局部变量来打破这种循环引用

注意：`CoreFoundation`对象不归 ARC 管理，开发者必须适时调用`CFRetain/CFRelease`


###在`dealloc`方法中只释放引用并解除监听

把原来配置过的观测行为都清除掉，如果使用`NSNotificationCenter`给此对象注册过某种通知，那么一般应该在这里注销

###使用弱引用来避免循环引用


###理解`Block`

如果`block`所捕获的变量是对象类型，那么就会自动保留它。系统在释放这个块的时候，也会将其一并释放。这引出一个重要问题。`block`块本身可视为对象，也有引用计数。

如果将`block`块定义在实例方法中，那么除了可以访问类的所有实例变量之外，还可以使用 self
变量，块总能修改实例变量，所以在声明时无需加`__block`。不过，如果通过读取或者写入操作捕获了实例比那两，那么也会自动把`self`变量一并捕获了，因为实例变量是与`self`所指代的实例关联在一起的。

**全局块**

定义块的时候，占的内存区域是分配在*栈*中的

给块发送`copy`消息拷贝，这样就可以把块从栈复制到堆了。

全局块不会捕捉任何状态，运行时也无须有状态来参与。

###为常用的块类型创建`typedef`


###用`handler`块降低代码分散程度


###使用`block`块引用所属对象不要出现引用循环

###多用派发队列，少用同步锁

`@synchronized(self)`根据给定的对象，自动创建一个锁，并等待块中的代码执行完毕

`NSLock`

不过最好使用 GCD，它能更简单，更搞笑的形式为代码加锁

使用**串行同步队列**，将读取操作以及写入操作都安排在同一个队列里，可保证数据同步

    _syncQueue = dispatch_queue_create("com.xx", NULL);
    
    dispatch_sync(_syncQueue, ^{
        _someString = someString;
    });
    
把设置和获取操作都安排在序列化的队列里执行，这样的花所有针对属性的访问操作都同步了

`dispatch_barrier_async(dispatch_queue_t queue,disaptch_block_t block);`在`barrier`中必须单独执行，不能与其他块并行

###多使用 GCD，少用`performSelector`系列方法

延后执行某个任务

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [self doSomething];
});
```

想把任务放在主线程上执行

    dispatch_async(dispatch_get_main_queue(), ^{
        [self doSomething];
    });


###掌握 GCD 及操作队列的使用时机

在执行后台任务时，GCD 并不一定是最佳方式，还有一种技术叫做`NSOperationQueue`

比如，从服务器端下载并处理文件的动作，可以用操作来表示，而在处理其他文件之前，必须先下载清单文件，后续的下载操作，都要依赖于先下载清单文件这一操作。如果操作队列允许并发的话，那么后续的多个下载操作就可以同时执行，但前提是它们所依赖的那个清单文件下载操作已经执行完毕

`NSOperation`对象有许多属性都适合 KVO 来监听，`isCancelled`来判断任务是否取消，或者通过`isFinished`来判断任务是否完成。

`NSOperation`对象也有线程优先级

###通过`dispatch group`机制，根据系统资源状况来执行任务


###使用`dispatch_once`来执行只需一次的线程安全代码

单例使用

###不要使用`dispatch_get_current_queue`

###熟悉系统框架

`CFNetwork`，此框架提供了 C 语言级别的网络通信能力，它将 BSD 抽象成易于使用的网络接口。而 Foundation 则将该框架李的部分内容封装为 OC 的接口

`CoreAudio`，此框架提供的 C 语言 API 可以用来操作设备商的音频硬件

`AVFoundation`，用来回访并录制音频及视频

`CoreData`，将对象放入数据库

`CoreText`，可以高效执行文字排版及渲染操作


###多用块枚举，少用 for 循环


###构建缓存时选用`NSCache`而非`NSDictionary`


###精简`initialize`与`load`的实现代码

`+ (void)load`，只调用一次。

`+ (void)initialize`，该方法在程序首次使用该类之前调用，且只调用一次


###`NSTimer`会保留其目标对象

NSTimer很容易出现引用循环

（未完待续）

#总结
纯属个人笔记，特别是底层机制很有作用，如今`iOS`开发不再仅仅是把一个内容展现出来，里面还有涉及到各种安全性能，了解根本才是持续发展之道。
