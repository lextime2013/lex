#### Eclipse项目迁移到Android Studio的一种方式

##### 题外话

不知道你们公司是不是还停留在eclipse IDE呢，是不是觉得由于项目依赖太麻烦就一直没迁移过去呢。Android Studio 2.0 都发布第三版预览版本了，Google也早已停止了对ADT插件的更新。

但将老项目迁移过去确实有点麻烦，各种项目依赖、各种gradle build…什么的导致崩溃卡机。我已被虐过千百遍，但依然对它如初恋。我实践了一种方式，下面主要讲一下大体思路，对gradle完全不熟悉的话可能会被省去了一些细节。

##### 1、基本准备

下载android studio，我这里用android studio1.3、1.4都试过， 
新建一个工程，都是默认，确保这个工程能运行。然后看看这个工程文件，目录结构 
这里写图片描述 
.gradle、.idea、build、都是运行时产生，删除。重要的是gradle文件夹，gradlew，gradlew.bat，build.gradle，setting.gradle。然后熟悉其他文件是干嘛用的，是否删除或修改取决于你原来的eclipse项目。

##### 2、新建文件夹然后copy工作

新建一个文件夹，按照上面MyApplication的目录结构，copy一份，其中需要从eclipse项目中迁移过来的有src源代码，还有res资源，注意修改主工程build.gradle里面的applicationId。然后用Android Studio打开新建的项目即可。

##### 3、如何解决项目依赖

很多情况下，并不想把原来的project变成一个module，所以有了工程中的module依赖。这里需要手动配置setting.gradle文件，添加

include ':projectName'
project(':projectName').projectDir = new File(settingsDir, '../projectName/moduleName')
1
2
1
2
然后在主工程的module的build.gradle文件里面添加

dependencies {
    compile project(':projectName')
}
1
2
3
1
2
3
同步一下，这里就实现了对工程projectName中的module依赖。 
可以参考： 
http://www.cnblogs.com/avenwu/p/4299340.html

##### 遇到的一些问题

1、Android6.0在api 23以上没有了apache的包，也就是没有了HttpClient等网络相关的类，需要使用的可以从以下路径找到，

**\android-sdk-windows\platforms\android-23\optional
1
1
copy到工程的libs里面，并在build.gradle里面添加：

useLibrary 'org.apache.http.legacy'
1
1
2、主工程module的build.gradle里面是

apply plugin: 'com.android.application'
1
1
依赖工程module的build.gradle里面应该是依赖库的形式

apply plugin: 'com.android.library'
1
1
3、依赖工程里面不需要applicationId。 
4、androidManifest.xml里面的appIication icon冲突可以删除或修改，错误日志：

Error:Execution failed for task ':app:processDebugManifest'.
> Manifest merger failed : Attribute application@icon value=(@drawable/logo) from AndroidManifest.xml:49:9-38
 is also present at
 ...
1
2
3
4
1
2
3
4
5、引用重复的jar包，删除其中一个，错误日志：

Error:Execution failed for task ':app:dexDebug'.
> com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException: Process 'command 'D:\Program Files\Java\jdk1.7.0_80\bin\java.exe'' finished with non-zero exit value 2
1
2
1
2
6、错误日志：

Error:Execution failed for task ':app:packageDebug'.
> Duplicate files copied in APK META-INF/LICENSE
...
1
2
3
1
2
3
按照提示，添加：

android {
    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
    }
}
1
2
3
4
5
6
7
8
9
1
2
3
4
5
6
7
8
9
总结

也就是将标准Android Studio工程里面的gradle相关文件原汁原味的copy到另外一个新建的、具有相同目录结构的文件夹里面，做一些相应的配置修改。打开Android Studio运行。

##### 为什么使用这种方式？

网上比较多的教程是eclipse导出gradle项目，生成build.gradle文件，再用android studio导入，我会觉得这个项目有点奇葩，各种ant、gradle构建的东西混杂在一起（对不起，我有代码洁癖），还会因为gradle构建版本的各种问题而运行不起来，而且项目依赖也并不怎么清晰。或许还有更简便的方法，我在瞎折腾了。

##### 最后

最后想说明一点，这是其中一种方式，我实践可用，记录下来一方面自己学习温故，另一方面希望能帮到一些人，这个方法或许有不足之处，希望指正。gradle配置方面还有很多，有些没涉及到的可能还需要添加其他的配置代码。