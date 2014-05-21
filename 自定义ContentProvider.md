［转］http://blog.csdn.net/sadfishsc/article/details/7419573
适用范围
-----------------------------
        对于什么情况下才会用到自定义的ContentProvider，官方文档的Dev Guide是这样描述的：
    如果你想要提供以下的一种或几种特性的时候你才需要构造一个ContentProvider：
        * 你想要为其它的应用提供复杂的数据或者文件；
        * 你想允许用户从你的应用中拷贝复杂的数据到其它的应用中；
        * 你想要使用搜索框架来提供自定义的搜索策略；
        * 你完全不需要ContentProvider来调用一个SQLite数据库，如果这种调用完全在你自己的应用之中。
    也就是说，ContentProvider的作用是为别的应用调用本应用中的数据或者文件提供接口，而它也是唯
    一的跨应用数据传递的接口(注：AIDL也能通过接口实现传递数据)。如果仅仅是同一个应用中的数据传
    递，则完全没有必要使用到自定义的ContentProvider。另一方面，虽然ContentProvider也能组织
    文件数据或者SharedPreferences(其实也是文件数据）这种数据，但大多数情况下ContentProvider
    是作为SQLite数据库的调用接口来被继承的。其原因大概是在于重写的query()方法始终需要返回Cursor，
    而Cursor作为数据库数据的容器，并没有提供直接往Cursor中写入数据的方法。

实现步骤
-----------------------------
    * 创建一个数据源，例如继承SQLiteOpenHelper创建一个SQLite数据库；

    * 创建一个继承自ContentProvider的类，并重写insert、delete、query、update、getType、onCreate方法，在这些方法中实现对数据源的操作；

    * 在AndroidManifest.xml文件中添加<provider>标签，两个必写的属性是android:name和android:authorities；

    * 在本应用或者其它应用的Activity、Service等组件中使用ContentResolver通过对应的URI来操作该自定义ContentProvider。

    * URI:Android各种类型的URI基本上都是有固定格式的，对于ContentProvider而言，一般形如
        ** content://com.test.cp.MyProvider/phone/1的URI，其中：
            *** content://是固定字段，必需；
            *** com.test.cp.MyProvider表示authority，是AndroidManifest.xml文件中<provider>标签的android:authorities属性值，或者是远程数据源的主机名，必需；
            *** phone/1表示path，是数据源路径，非必需，其中的phone对于数据库来说可以视为表名，1表示的是该条数据的编号，如果没有则一般认为是返回当前路径（当前表）中的所有数据。
                另外还可以根据自己的需要来进一步定义后续的字段。

    * onCreate方法与构造方法ContentProvider没有显式地执行初始化的语句，因此即便是重写了它的构造方法也不会被执行。它的初始化代码一般都写在onCreate方法中。但是网上的例子中也有部分初始化代码被写在了静态域之中（主要是关于UriMatcher的初始化代码）。不过经过本人测试发现，把这些放在静态域中的代码移到onCreate方法中也不会影响程序的运行。
      另外需要注意的是必须把onCreate方法的返回值该为true，该ContentProvider才能被加载。

    * UriMatch对象
      UriMatch对象的作用是将URI匹配到对应的表（就数据库而言），其使用步骤如下：
        * 通过new  UriMatcher(UriMatcher.NO_MATCH); 实例化，常量NO_MATCH作为参数表示不匹配任何URI；
        * 实例化后调用addURI方法注册URI，该方法有三个参数，分别需要传入URI字符串的authority部分、path部分以及自定义的整数code三者；
        * 其它地方调用match方法匹配相应的URI，需要传入Uri作为唯一的参数，返回上述自定义的code值。
          至于其初始化的位置，如前所述，网上绝大多数示例都将其放入静态域中实例化，原因不明。实际上放到onCreate方法中也没什么问题。

    * getType方法ContentProvider必须重写的6个方法中，除了初始化方法onCreate以及数据操作的4个方法以外，还有一个getType方法。它的作用是根据URI返回该URI所对应的数据的MIME类型字符串。这种字符串的格式分为两段：“A/B”。其中A段是固定的，集合类型（如多条数据）必须是vnd.android.cursor.dir，非集合类型（如单条数据）必须是vnd.android.cursor.item；B段可以是自定义的任意字符串；A、B两段通过“/”隔开。这个MIME类型字符串的作用是要匹配AndroidManifest.xml文件<activity>标签下<intent-filter>标签的子标签<data>的属性android:mimeType。如果不一致，则会导致对应的Activity无法启动。

    * notifyChange方法
        网上的某些示例中在重写insert、delete、update、query方法对数据的操作结束之后，总会加一句代码：
        getContext().getContentResolver().notifyChange(uri,null);
        其作用是通知在ContentResolver中注册了该URI的ContentObserver，这个URI对应的数据源发生变化了。其具体用法参见下面的链接：
        http://blog.csdn.net/zhf198909/article/details/6903708
        另外，通知变化对于ContentProvider来说并不是必需的，根据实际功能的需要，自定义的ContentProvider中多数情况下并不需要这句代码。

    * 示例
        http://www.cnblogs.com/chenglong/articles/1892029.html
