看开源代码时，总会看到一些大神级别的代码，给人眼前一亮的感觉，多数都是被淡忘的C语言语法，总结下objc写码中遇到的各类`非主流`代码技巧和一些妙用：

* [娱乐向]objc最短的方法声明
* [C]结构体的初始化
* [C]三元条件表达式的两元使用
* [C]数组的下标初始化
* [objc]可变参数类型的block
* [objc]readonly属性支持扩展的写法
* [C]小括号内联复合表达式
* [娱乐向]奇葩的C函数写法
* [Macro]预处理时计算可变参数个数
* [Macro]预处理断言
* [多重]带自动提示的keypath宏

---

##[娱乐向]objc最短的方法声明")[娱乐向]objc最短的方法声明

先来个娱乐向的。
方法声明时有一下几个trick： 

返回值的`- (TYPE)`如果不写括号，编译器默认认为是`- (id)`类型:

    - init;
    - (id)init; // 等价于

同理，参数如果不写类型默认也是`id`类型: 

    - (void)foo:arg;
    - (void)foo:(id)arg; // 等价于

还有，有多参数时`方法名`和`参数提示语`可以为空

    - (void):(id)arg1 :(id)arg2;
    - (void)foo:(id)arg1 bar:(id)arg2; // 省略前

综上，最短的函数可以写成这样：

    - \_; // 没错，这是一个oc方法声明
    - :\_; // 这是一个带一个参数的oc方法声明
    // 等价于
    - (id)\_;
    - (id) :(id)\_;

*PS: 方法名都没的方法只能靠`performSelector`来调用了，`selector`是`":"`*

---

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#C-%E7%BB%93%E6%9E%84%E4%BD%93%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96 "[C]结构体的初始化")[C]结构体的初始化

    // 不加(CGRect)强转也不会warning
    CGRect rect1 = {1, 2, 3, 4};
    CGRect rect2 = {.origin.x=5, .size={10, 10}}; // {5, 0, 10, 10}
    CGRect rect3 = {1, 2}; // {1, 2, 0, 0}

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#C-%E4%B8%89%E5%85%83%E6%9D%A1%E4%BB%B6%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%9A%84%E4%B8%A4%E5%85%83%E4%BD%BF%E7%94%A8 "[C]三元条件表达式的两元使用")[C]三元条件表达式的两元使用

三元条件表达式`?:`是C中唯一一个三目运算符，用来替代简单的`if-else`语句，同时也是可以**两元**使用的：

    NSString \*string = inputString ?: @"default";
    NSString \*string = inputString ? inputString : @"default"; // 等价

利用这个特性，我们还脑洞出了一个一行代码的 block 调用，平时我们的 block 是这样调用：

    if (block0) {
     block0();
    }
    // or
    if (block1) {
     int result = block1(1, 2);
    }

居然可以简化成下面的样子：

    !block0 ?: block0();
    int result = !block1 ?: block1(1, 2);

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#C-%E6%95%B0%E7%BB%84%E7%9A%84%E4%B8%8B%E6%A0%87%E5%88%9D%E5%A7%8B%E5%8C%96 "[C]数组的下标初始化")[C]数组的下标初始化

    const int numbers[] = {
     [1] = 3,
     [2] = 2,
     [3] = 1,
     [5] = 12306
    };
    // {0, 3, 2, 1, 0, 12306}

这个特性可以用来做`枚举值和字符串的映射`

    typedef NS\_ENUM(NSInteger, XXType){
     XXType1,
     XXType2
    };
    const NSString \*XXTypeNameMapping[] = {
     [XXType1] = @"Type1",
     [XXType2] = @"Type2"
    };

---

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#objc-%E5%8F%AF%E5%8F%98%E5%8F%82%E6%95%B0%E7%B1%BB%E5%9E%8B%E7%9A%84block "[objc]可变参数类型的block")[objc]可变参数类型的block

一个block像下面一样声明：

    void(^block1)(void);
    void(^block2)(int a);
    void(^block3)(NSNumber \*a, NSString \*b);

**如果block的参数列表为空的话，相当于可变参数（不是void）**

    void(^block)(); // 返回值为void，参数可变的block
    block = block1; // 正常
    block = block2; // 正常
    block = block3; // 正常
    block(@1, @"string"); // 对应上面的block3
    block(@1); // block3的第一个参数为@1，第二个为nil

这样，block的主调和回调之间可以通过`约定`来决定block回传回来的参数是什么，有几个。如一个对网络层的调用： 

    - (void)requestDataWithApi:(NSInteger)api block:(void(^)())block {
     if (api == 0) {
     block(1, 2);
     }
     else if (api == 1) {
     block(@"1", @2, @[@"3", @"4", @"5"]);
     }
    }

主调者知道自己请求的是哪个Api，那么根据`约定`，他就知道block里面应该接受哪几个参数：

    [server requestDataWithApi:0 block:^(NSInteger a, NSInteger b){
     // ...
    }];
    [server requestDataWithApi:1 block:^(NSString \*s, NSNumber \*n, NSArray \*a){
     // ...
    }];

这个特性在`Reactive Cocoa`的`-combineLatest:reduce:`等类似方法中已经使用的相当好了。

    + (RACSignal \*)combineLatest:(id\<NSFastEnumeration\>)signals reduce:(id (^)())reduceBlock;

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#objc-readonly%E5%B1%9E%E6%80%A7%E6%94%AF%E6%8C%81%E6%89%A9%E5%B1%95%E7%9A%84%E5%86%99%E6%B3%95 "[objc]readonly属性支持扩展的写法")[objc]readonly属性支持扩展的写法

假如一个类有一个`readonly`属性：

    @interface Sark : NSObject
    @property (nonatomic, readonly) NSArray \*friends;
    @end

`.m`中可以使用`_friends`来使用自动合成的这个变量，但假如：

* 习惯使用`self.`来set实例变量时（只合成了getter）
* 希望重写getter进行懒加载时（重写getter时则不会生成下划线的变量，除非手动`@synthesize`）
* 允许子类重载这个属性来修改它时（编译报错属性修饰符不匹配）

这种`readonly`声明方法就行不通了，所以下面的写法更有通用性：

    @interface Sark : NSObject
    @property (nonatomic, readonly, copy/\*加上setter属性修饰符\*/) NSArray \*friends;
    @end

如想在`.m`中像正常属性一样使用：

    @interface Sark ()
    @property (nonatomic, copy) NSArray \*friends;
    @end

子类化时同理。iOS SDK中很多地方都用到了这个特性。 

---

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#C-%E5%B0%8F%E6%8B%AC%E5%8F%B7%E5%86%85%E8%81%94%E5%A4%8D%E5%90%88%E8%A1%A8%E8%BE%BE%E5%BC%8F "[C]小括号内联复合表达式")[C]小括号内联复合表达式

`A compound statement enclosed in parentheses`原谅我的渣翻译- -，来自[《gcc官方对此的说明》](https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html)，源自gcc对c的扩展，如今被clang继承。 

    RETURN\_VALUE\_RECEIVER = {(
     // Do whatever you want
     RETURN\_VALUE; // 返回值
    )};

于是乎可以发挥想象力了：

    self.backgroundView = ({
     UIView \*view = [[UIView alloc] initWithFrame:self.view.bounds];
     view.backgroundColor = [UIColor redColor];
     view.alpha = 0.8f;
     view;
    });

有点像block和内联函数的结合体，它最大的意义在于将代码`整理分块`，将同一个逻辑层级的代码包在一起；同时对于一个无需复用小段逻辑，也免去了重量级的调用函数，如：

    self.result = ({
     double result = 0;
     for (int i = 0; i \<= M\_2\_PI; i+= M\_PI\_4) {
     result += sin(i);
     }
     result;
    });

这样使得代码量增大时层次仍然能比较明确。 

*PS: 返回值和代码块结束点必须在结尾*

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#%E5%A8%B1%E4%B9%90%E5%90%91-%E5%A5%87%E8%91%A9%E7%9A%84C%E5%87%BD%E6%95%B0%E5%86%99%E6%B3%95 "[娱乐向]奇葩的C函数写法")[娱乐向]奇葩的C函数写法

正常编译执行： 

    int sum(a,b)
    int a; int b;
    {
     return a + b;
    }

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#Macro-%E9%A2%84%E5%A4%84%E7%90%86%E6%97%B6%E8%AE%A1%E7%AE%97%E5%8F%AF%E5%8F%98%E5%8F%82%E6%95%B0%E4%B8%AA%E6%95%B0 "[Macro]预处理时计算可变参数个数")[Macro]预处理时计算可变参数个数

    \#define COUNT\_PARMS2(\_a1, \_a2, \_a3, \_a4, \_a5, RESULT, ...) RESULT
    \#define COUNT\_PARMS(...) COUNT\_PARMS2(\_\_VA\_ARGS\_\_, 5, 4, 3, 2, 1)
    int count = COUNT\_PARMS(1,2,3); // 预处理时count==3

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#Macro-%E9%A2%84%E5%A4%84%E7%90%86%E6%96%AD%E8%A8%80 "[Macro]预处理断言")[Macro]预处理断言

下面的断言在编译前就生效 

    \#define C\_ASSERT(test) \\
     switch(0) {\\
     case 0:\\
     case test:;\\
     }

如断言上面预处理时计算可变参数个数： 

    C\_ASSERT(COUNT\_PARMS(1,2,3) == 2);

如果断言失败，相当于`switch-case`中出现了两个`case:0`，则编译报错。 

## [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#%E5%A4%9A%E9%87%8D-%E5%B8%A6%E8%87%AA%E5%8A%A8%E6%8F%90%E7%A4%BA%E7%9A%84keypath%E5%AE%8F "[多重]带自动提示的keypath宏")[多重]带自动提示的keypath宏

源自`Reactive Cocoa`中的宏： 

    \#define keypath2(OBJ, PATH) \\
     (((void)(NO && ((void)OBJ.PATH, NO)), \# PATH))

原来写过一篇[《介绍RAC宏的文章》](http://blog.sunnyxx.com/2014/03/06/rac_1_macros/)中曾经写过。这个宏在写PATH参数的同时是带自动提示的：
![](resources/77AEABF040D9D9ED2E50A781BA19A4FA.jpg)

### [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#%E9%80%97%E5%8F%B7%E8%A1%A8%E8%BE%BE%E5%BC%8F "逗号表达式")逗号表达式

逗号表达式取后值，但前值的表达式参与运算，可用void忽略编译器警告

    int a = ((void)(1+2), 2); // a == 2

于是上面的keypath宏的输出结果是`#PATH`也就是一个c字符串

### [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/#%E9%80%BB%E8%BE%91%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84 "逻辑最短路径")逻辑最短路径

之前的文章没有弄清上面宏中`NO&&NO`的含义，其实这用到了编译器优化的特性： 

```text
if (NO && [self shouldDo]/*不执行*/) {
    // 不执行
}

```

编译器知道在NO后且什么的结果都是NO，于是后面的语句被优化掉了。也就是说keypath宏中这个`NO && ((void)OBJ.PATH, NO)`就使得在编译后后面的部分不出现在最后的代码中，于是乎既实现了keypath的自动提示功能，又保证编译后不执行多余的代码。 

---

# [](http://blog.sunnyxx.com/2014/08/02/objc-weird-code/ "References")References