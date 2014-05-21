#[转]iOS应用程序的状态及其切换（生命周期）

标签（空格分隔）： ios


---
【原文】http://www.molotang.com/articles/1254.html
    做iOS开发有小一年了，做应用开发有好几年了。大大小小的应用也开发了很多种，但无论是运行在哪个平台上的用什么语言开发出来的应用程序，对于执行内容比较长的、场景比较复杂的情况，通过状态和生![此处输入图片的描述][1]命周期来组织代码是非常好的一种方式。本文对iOS应用涉及到的生命周期，或者叫应用程序状态做一个简要的小结。
    刚刚提到生命周期在应用程序开发中的应用很广泛，我了解到的例子就有很多。比如Java中的Tomcat实现，无论是Server、Service还是Connector或者Context，都是Lifecycle的子类实现；再如JDK1.5之后的ThreadPoolExecutor也是有状态概念的；往近了说，除了iOS应用外，与其呼应的Android应用中，Activity也是有生命周期概念的。
    说得有点远了，书归正传，说说iOS应用程序中的状态，就从应用启动说起。
    

 ##  iOS应用入口和AppDelegate ##
我最初开始iOS应用开发学习的时候，也没有像样的培训和指导，第一个要看的就是代码。我们知道ObjectiveC也是基于C的，于是我们找到了入口代码main()函数。通常代码如下：

    int main(int argc, char *argv[])
    {
       @autoreleasepool {
          return UIApplicationMain(argc, argv, nil, NSStringFromClass([MyAppDelegate class]));
       }
    }

我们看到返回类型、函数名和参数都很熟悉。至于函数体，除了AutoRelease是自动释放池之外，执行的是一个名为UIApplicationMain的函数调用，后两个参数是nil（空）和一个MyAppDelegate类名字符串。这个函数调用也就是这个iOS应用的开始，它进行了这个应用的初始化过程，并生成了一个类名为MyAppDelegate的对象。至于MyAppDelegate这个名字，这里只是一个示例，可以为其它命名，通常是应用统一定义的前缀+”AppDelegate”。这些都不重要，重要的是它实际上需要遵从于UIApplicationDelegate这个Protocol的定义，给出一些方法的实现。那么这个“AppDelegate”又是什么呢？实际上，苹果已经为把每个应用包装成一个UIApplication对象，但应用每一步运行的细节并不需要开发者关注，只要关注这个应用对象对应的delegate即可，也就是这个“AppDelegate”。通过AppDelegate我们可以知道Application的运行状态，发生了哪些事件。 

 - 程序的5个状态和对应的AppDelegate的7个方法
 至于五个状态，分别是：

     - Not Running, 
     - 未运行 Inactive,
     - 非活动  Active,
     - 活动 Background,
     - 后台 Suspend,挂起

对这5种状态，这里先不过多解释，看下图也许就会明白许多。


  从这个示意图，我们可以看到哪些状态间是可以互相转化的。而在这些状态互相转化的同时，AppDelegate中对应的生命周期方法会被调用：
  从这个示意图，我们可以看到哪些状态间是可以互相转化的。而在这些状态互相转化的同时，AppDelegate中对应的生命周期方法会被调用：

    - (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions      进程启动但还没完成初始化
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions     进程启动基本完成
    - (void)applicationWillResignActive:(UIApplication *)application    应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件
    - (void)applicationDidBecomeActive:(UIApplication *)application    应用程序入活动状态，这个刚好跟上面那个方法相反
    - (void)applicationDidEnterBackground:(UIApplication *)application     程序被推送到后台，如果要设置后台继续运行，则在这个函数里面设置即可
    - (void)applicationWillEnterForeground:(UIApplication *)application   程序从后台将要回到前台
    - (void)applicationWillTerminate:(UIApplication *)application   程序将要退出
## 应用一般启动过程 ##
介绍了iOS应用的基本状态和生命周期方法之后，再简要对应用启动和前台状态做一个整理说明。

对于一般的iOS应用来说，启动之后是进入前台运行的，大概的流程如下图：

![应用启动过程][2]
也就是说通常情况下，应用程序启动，图中的三个方法会被依次调用到。

其中前两个方法是我们需要做一些初始化的地方，只在启动时调用，而后面的BecomeActive方法会因程序中断和前后台切换多次调用到。

对于一个简单的应用，初始化主要做的事情就是把View和Controller初始化并结合起来。此外，必要的情况下还要对传入的Options参数进行解析处理，构建必要的数据结构。而按照官方文档的要求，整个启动过程需要在5秒钟内完成，否则应用进程会因无响应的原因被kill掉。因此，如果有其它任务需要执行，则应该开启主线程之外的线程来进行，或者是推迟到之后更合适的实际来进行。

应用程序也有启动后直接进入后台运作的，这个可以参看苹果官方文档说明。
##  前台运行的Active与Inactive ##
在介绍iOS应用状态5种最基本的状态时，我们发现前台运行有两种状态，分别是Inactive和Active状态。大多数情况下，Inactive状态只是其它状态之间切换时短暂的停留状态，如前后台应用切换时，Inactive状态会在Active和Background之间短暂出现。

但也有一些其它情况，Active和Inactive可以在前台运行时互相切换，比如当一个应用安装运行后第一次尝试使用GPS定位，需要获取用户的允许，给出系统的Alert提示，这时应用会从Active切换到Inactive，直到用户确认后再返回Active。再如，用户在应用运行时从状态条向下拉出通知页，也会发生Active和Inactive状态的切换。
![前台运行的Active与Inactive #][3]
## 参考 ##

有关前后台状态切换、iOS应用在后台运行、多任务并发和事件循环的内容留在后续文正整理总结。

本篇文章是本人对官方文档学习和简单实践之后自己的一些理解，如有纰缪还请多多指出。

参考文档：[`App States and Multitasking`][4]


  [1]: https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/high_level_flow_2x.png
  [2]: https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/app_launch_fg_2x.png
  [3]: https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/app_interruptions_2x.png
  [4]: https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ManagingYourApplicationsFlow/ManagingYourApplicationsFlow.html#//apple_ref/doc/uid/TP40007072-CH4-SW3
