---
title: Android模块化知识分享
date: 2021-08-08 15:09:42 +/-TTT
categories: [Tech, Android]
tags: [android, modular]
---
# 1 Android模块化知识分享
> 未经允许，禁止转载
## 1.1 定义
### 1.1.1 模块化定义
> **Modular programming** is a software design technique that emphasizes separating the functionality of a program into independent, interchangeable **modules**, such that each contains everything necessary to execute only one aspect of the desired functionality.（摘自wiki百科）
>
> 模块化编程是一种软件设计技术，它强调将程序的功能分离为独立的、可互换的模块，使每个模块都包含执行所需功能的一个方面所必需的一切。

大致意思就是，模块化将一个程序按照**不同的功能**拆分为**相互独立**的模块，每个模块只负责自己业务的开发，最后再将这些模块集成到**主模块(Application Module)** 中，通过路由实现不同模块间的跳转、通信和解耦。
### 1.1.2 组件化开发定义
> **Component-based software engineering (CBSE)** , also called **components-based development (CBD)** , is a branch of software engineering that emphasizes the separation of concerns with respect to the wide-ranging functionality available throughout a given software system.（摘自wiki百科）
>
> 基于组件的软件工程（CBSE），也称为基于组件的开发（CBD），是软件工程的一个分支，它强调在给定的软件系统中，对于通用功能进行分离的思想。

事实上，模块化与组件化之间没有明显的界限，组件化就是对不同模块的进一步细分，将一些模块的**通用功能**剥离出来形成一个组件，进一步提高程序的复用性。
## 1.2 模块化与组件化的区别

-   模块化是**业务导向**，主要针对业务逻辑层的拆分
-   组件化是**功能导向**，主要针对项目功能的拆分和重用

## 1.3 模块化/组件化的优势

-   单独编译，独立运行，减少项目编译运行时间
-   项目结构清晰，便于维护，提高团队开发效率
-   业务功能分离，实现高内聚，低耦合

## 1.4 项目架构的演变
这里以我最近练手的项目为例进行说明
### 1.4.1 无架构阶段

<div align=center>
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60cd6cc1452d4e2aafebfccef180b6b9~tplv-k3u1fbpfcp-watermark.image" width="65%;"/>
</div>

如果项目的所有功能都堆叠到一个模块中，代码间的耦合度就会大大提高，而且每当我们修改完一个功能的代码后就需要运行整个项目，等待时间长。

### 1.4.2 模块化阶段

<div align=center>
<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09753613bbb046ee83b5684fd6445556~tplv-k3u1fbpfcp-watermark.image" width="65%;"/>
</div>

按照业务模块进行拆分每个业务都是一个**module**，降低耦合度且能实现单独编译运行。

### 1.4.3 业务层分离

<div align=center>
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89c7af184fee4b4eb5c03006edcc668b~tplv-k3u1fbpfcp-watermark.image" width="80%;"/>
</div>

将业务层模块中通用的方法提取出来形成**基础层组件lib_base**，提高功能的复用性，进一步解耦。这里我们分别使用`module_`和`lib_`前缀来区分业务组件和功能组件。

### 1.4.4 基础层分离

<div align=center>
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b23d3033e68e495b96835e3e6c23777f~tplv-k3u1fbpfcp-watermark.image" width="80%;"/>
</div>

如果一些自定义view，布局文件或者资源文件等会被多个模块使用，那么应该将它们从中剥离出来。但是如果将它们放到module_base中会破坏基础层的通用性，所以需要一个**公共层module_common**来专门提供上层业务模块的公共资源，同时也保证了**基础层代码的通用性**，不管放到哪个项目里面都可以正常运行。
## 1.5 模块化/组件化实战——从问题出发

### 1.5.1 如何实现模块/组件的单独调试？

#### 1.5.1.1 实现效果

app主模块和文章模块分别编译运行后的结果如图所示：

<div align=center>
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/517576062fc14301a4a47b94cdb5fcf5~tplv-k3u1fbpfcp-watermark.image" width="40%;"/>
</div>

其实要实现单独编译运行很简单，一共只需要**两步**。

#### 1.5.1.2 配置不同的插件

在没有模块化项目前，我们通常只有一个app主模块，在app模块的`build.gradle`文件中通常第一行都会是下面这行代码：

> apply plugin: 'com.android.application'

这行代码的意思是指，在app模块中使用Android Gradle为我们提供的**Application插件**，其中`'com.android.application'`是该插件的id。Android Gradle一共为开发者提供了三种不同的插件，来帮助我们在开发过程中通过配置不同的插件来配置不同的工程：

-   App插件：com.android.application
-   Library插件：com.android.library
-   Test插件：com.android.test

其中使用了App插件的模块**可以被独立运行**，通常被用在可独立调试的业务模块中，如`module_article`, `module_user`等；使用了Library插件的模块**不可独立运行**，一般用在项目依赖的组件中，如`lib_common`, `lib_base`。

那么通过上面的知识，其实我们就已经有了独立调试模块的大概思路了，只是这个思路还需要一些完善。

首先我们需要注意的是，在组件化开发过程中，各个业务组件既可以**单独调试**又可以**被其他模块依赖**，这就需要我们提供**一个变量`isSingleModuleDebug`来判断**当前要运行的程序是否为模块的单独调试状态，如果为true，则各个组件可以被单独编译运行，反之则不行。有了思路，接下来我们就直接将其转换为代码吧。

首先在根目录下的`gradle.properties`文件中添加一个变量`isSingleModuleDebug`：

(Tips：`gradle.properties`文件中的内容可以被全局使用)

```gradle
# 模块是否独立运行
isSingleModuleDebug=true
```

然后打开所有需要单独调试模块的`build.gradle`文件（例如`module_article`），在最上面添加以下判断代码并删除之前的`apply plugin: 'com.android.application'`：

```gradle
if (isSingleModuleDebug.toBoolean()){
    apply plugin: 'com.android.application'
}else{
    apply plugin: 'com.android.library'
}
```

如果需要单独调试，则使用App插件，否则就使用Library插件（其实到每个模块中去挨个添加有些麻烦，后面会有一些优化的内容）。

另外，不要忘了删除**除了主模块以外**其他模块`build.gradle`文件中的`applicationId`，**在集成调试时项目只允许出现一个`applicationId`**。

```gradle
android {
    defaultConfig {
        // 删除掉这一行
        applicationId "com.aefottt.module_article"
    }
}
```

#### 1.5.1.3 配置AndroidManifest文件

在新建一个模块后您会发现，在该模块的`manifests/AndroidManifest.xml`文件中存在以下代码，代表着在启动app时会以ArticleActivity文件为启动页：

```xml
<activity android:name=".ArticleActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

当然，如果是单独运行该模块这段代码不会有啥问题，但是当各模块**集成调试**的时候呢？每个模块都有一个启动页，那app在启动时该先启动哪个界面？

针对这个问题，我们需要配置两个`AndroidManifest.xml`文件，一个用于单独调试，一个用于集成调试，然后再在`build.gradle`文件中通过`isSingleModuleDebug`变量来判断并加载相应的`AndroidManifest.xml`文件。

首先，我们切换至AndroidStudio的Project目录：

<div align=center>
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e37611bd1a74484bc6f6e094c260ddd~tplv-k3u1fbpfcp-watermark.image" width="30%;"/>
</div>

在`module_article`模块的`src/main`文件夹下新建`module`文件夹，在`module`文件夹下新建`AndroidManifest.xml`文件。在模块单独调试运行时就会加载`module`文件夹下的`AndroidManifest.xml`文件：

<div align=center>
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b548a6aa9714c549a6e9cb20f94f484~tplv-k3u1fbpfcp-watermark.image" width="40%;"/>
</div>

然后将`main/AndroidManifest.xml`文件中的代码内容全部**搬运**到`module/AndroidManifest.xml`文件中，在`main/AndroidManifest.xml`文件中只保留以下代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.aefottt.module_article">

    <application android:theme="@style/AppTheme">
        <activity android:name=".ArticleActivity" />
    </application>

</manifest>
```

另外，您还需要注意的一点是，**在集成调试的时候所有模块中Application的主题必须一致**，否则在集中调试的时候会报错。由于所有模块都会引入`lib_common`库，我先在`lib_common`模块下的`values`文件中定义了一个通用的style: `AppTheme`，然后在所有模块的`main/AndroidManifest.xml`文件中设置application统一为该主题。当然您也可以使用其他模块的主题，只需要保证统一即可。

处理完`AndroidManifest.xml`文件的内容后我们就可以开始配置module_article下的`build.gradle`文件了：

```gradle
android{
    sourceSets{
        main{
            if (isSingleModuleDebug.toBoolean()){
                manifest.srcFile 'src/main/module/AndroidManifest.xml'
            }else {
                manifest.srcFile 'src/main/AndroidManifest.xml'
            }
        }
    }
}
```

代码的意思简单明了，如果是单独调试就加载module文件夹中的AndroidManifest文件，否则加载main下面的AndroidManifest文件，避免了集中调试时启动页的冲突问题。

接下来您就可以选择module_article模块，然后点击运行了。
<div align=center>
<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc92b540bf78496393f45f6b4e86eaa1~tplv-k3u1fbpfcp-watermark.image" width="60%;"/>
</div>

#### 1.5.1.4 配置文件优化 进一步解耦

在上面apply插件的时候已经提到过，如果一个模块一个模块的去添加判断会有些麻烦，这些都是模板代码，完全可以提取出来，这是第一个问题。第二个问题就是依赖库版本太乱，需要进行统一管理。针对这两个问题，我们需要在项目根目录下新建两个gradle文件：**config.gradle和version.gradle**，一个是所有模块`build.gradle`文件的统一模板，另一个是对项目构建、依赖等版本号的统一管理。

首先我们可以先观察一下所有模块通用的`build.gradle`文件是如何构成的。一共有三个作用域：plugin，android和dependencies。为了简单起见，我们直接全选任意模块的`build.gradle`内容，粘贴到`config.gradle`文件中去，将原本通过`isSingleModuleDebug`来判断并配置插件的代码换成下面的几行：

```gradle
if (isSingleModuleDebug.toBoolean()){
    if (project.name.matches("lib_.+"))
        apply plugin: 'com.android.library'
    else
        apply plugin: 'com.android.application'
}else {
    if (project.name != "app")
        apply plugin: 'com.android.library'
}
```

在单独调试时如果模块名以`lib_`开头，则配置`Library插件`，否则配置`App插件`。在集成调试时，除了`app主模块`，其他模块统一配置`Library插件`。

这里我们统一将**不需要单独调试的模块（即功能组件）** 前缀改为**lib_** ，这是因为在单独调试时功能组件不需要进行调试，而且一般在功能组件中也不存在Activity，所以功能组件在允许时需要一直使用`Library插件`，如果这里不加判断的话，在单独调试的时候功能组件会出现applicactionId的报错问题。

接下来，我们对`build.gradle`中对`AndroidManifest.xml`文件的内容进行提取，然后放到`config.gradle`文件中去：

```gradle
android{
    sourceSets{
        main{
            if (!project.name.matches("lib_.+")){
                if (isSingleModuleDebug.toBoolean()){
                    manifest.srcFile 'src/main/module/AndroidManifest.xml'
                }else {
                    manifest.srcFile 'src/main/AndroidManifest.xml'
                }
            }
        }
    }
}
```

如果模块的名字不以`lib_`开头，则在单独调试和集中调试时分别使用不同的`AndroidManifest.xml`文件。

这样之后，我们就可以把**除了主模块以外所有模块**的`build.gradle`文件内容全部删除，取而代之一行代码：

```gradle
apply from: '../config.gradle'
```

由于在集成调试的时候需要有`App插件`和`applicationId`，所以在主app模块的`build.gradle`文件中我们还需要几行内容：

```gradle
apply plugin: 'com.android.application'
apply from: '../config.gradle'

android {
    defaultConfig {
        applicationId "com.aefottt.gankio"
    }
}
```

第一个问题顺利解决，接下来我们看第二个问题，打开version.gradle文件，在这里对版本号进行统一管理。首先看一下项目版本和构建版本号的集中管理：

```gradle
/** 版本号管理 **/
ext.version_code = 1
ext.version_name = "1.0"
/** 构建版本管理 **/
def build_versions = [:]
build_versions.compile_sdk = 30
build_versions.build_tools = "30.0.3"
build_versions.min_sdk = 21
build_versions.target_sdk = 30
ext.build_versions = build_versions
```

`ext`是指在gradle中创建扩展插件，使得该属性能在其他gradle中被调用。

然后这里的`buile_versions`其实是一个HashMap，当然，即使不知道这个，之后的代码相信您也能很好的理解，为buile_versions添加相应的key-value键值对，最后将其添加到扩展插件中去。

然后为了能在其他gradle中使用`version.gradle`中的扩展插件，我们还需要打开项目**根目录**下的`buile.gradle`文件，在`buildscript{}`作用域中添加下面这行代码，表示全局可使用version.gradle中的插件：

```gradle
apply from: "version.gradle"
```

接下来我们就可以替换`config.gradle`文件中关于版本号的内容了：

```gradle
android {
    compileSdkVersion build_versions.compile_sdk
    buildToolsVersion build_versions.build_tools

    defaultConfig {
        minSdkVersion build_versions.min_sdk
        targetSdkVersion build_versions.target_sdk
        versionCode version_code
        versionName version_name

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    ......
}
```

至此对于两个问题的优化已经基本结束，事实上，您还可以对依赖库进行进一步的优化，例如将`config.gradle`中的版本库删除，取而代之将一些通用的依赖库添加到`lib_common`库中，然后在`config.gradle`中添加对`lib_common`的依赖。另外，您还可以将依赖库的版本号都放到`version.gradle`文件中，然后在使用**implementation/api**导入依赖，关于这部分代码您可以前往该项目的github地址（文章末尾处）查看。关于implementation和api的区别我在这里还需要提一下：

-   implementation：使用该指令的依赖将隐藏在模块内部，而不对外公开，可以加快编译速度。
-   api：等同于compile，完全公开依赖，使得依赖可以被依赖该模块的模块使用。

这里举个例子来帮助您更好的理解，假如在`lib_common`基础模块中导入了retrofit依赖，然后在`module_article`模块的`build.gradle`依赖中添加对`lib_common`的依赖，如果`lib_common`是使用api导入的retrofit依赖，就可以在`module_article`模块中使用Retrofit库，但如果是使用Implementation导入的，`module_article`就无法使用Retrofit库了。所以一般我们在`lib_common`中都使用api来导入依赖。

### 1.5.2 如何实现模块/组件间的通信？

（这一节内容主要参考任玉刚前辈的文章，文章链接会在结尾指出。）

#### 1.5.2.1 模块/组件间界面跳转

首先来看一下这样的业务场景：在启动APP后进入`启动页SplashActivity`，1秒后跳转，跳转时需要判断用户是否已登录，如果未登录则跳转到登录界面，反之跳转到主界面并加载用户信息。

<div align=center>
<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3403e4888e9c49f19549527b6ca04366~tplv-k3u1fbpfcp-watermark.image" width="58%;"/>
</div>

其中主界面和启动页在app主模块中，登录界面在module_user中，那么对于不同模块间的跳转在这里我们将使用**ARouter**来实现。

> ARouter是一个用于帮助 Android App 进行组件化改造的框架 —— 支持模块间的路由、通信、解耦。

ARouter可以帮助我们在不依赖彼此模块的前提下，实现不同模块间界面的跳转。使用ARouter一共分为三个步骤：**配置，初始化，添加路径**。

##### 一、配置

配置一共需要注意三个点：

1.  所有模块都需要引入arouter-api依赖，这里我们就在module_base(基础组件)的`build.gradle`中添加依赖：

    ```gradle
    api deps.arouter.api
    ```

    这里的`deps.arouter.api`是我在`version.gradle`中定义的统一管理依赖版本的扩展插件中的内容，您也可以像下面这样写：

    ```gradle
    api "com.alibaba:arouter-api:1.5.0"
    ```

2.  在**所有需要用到ARouter的模块中**添加arouter-compiler依赖，这里我为了方便就将其添加到了`config.gradle`（所有模块都会引入的gradle）中了：

    ```gradle
    kapt "com.alibaba:arouter-compiler:1.2.2"
    ```

3.  同样在**所有需要用到ARouter的模块中**，在android{}领域中添加`javaCompileOptions`配置，这里我也把他们统一放在`config.gradle`中了：

    ```gradle
    android{
        defaultConfig{
            ......
            javaCompileOptions {
                annotationProcessorOptions {
                    arguments = [ AROUTER_MODULE_NAME : project.getName() ]
                }
            }
        }
    }
    ```

##### 二、初始化

在app主模块中新建`GankApplication.kt`文件，继承自Application()，然后在`onCreate()`方法中添加如下几行代码：

```kotlin
    private val isDebug = false

    override fun onCreate() {
        super.onCreate()

        // 初始化ARouter
        if (isDebug){
            ARouter.openLog()
            ARouter.openDebug()
        }
        ARouter.init(this)
        
    }
```

最后别忘了在主模块的`AndroidManifest.xml`文件中设置`android:name=".GankApplication"`

##### 三、添加路径

以上面的业务场景为例，需要跳转的界面有两个：`MainActivity`和`LoginActivity`，分别找到其对应的文件，在最外层class的上方添加路径：

```kotlin
@Route(path="/main/main")
class MainActivity : AppCompatActivity() {
    ......
}
```

```kotlin
@Route(path="/user/login")
class LoginActivity : AppCompatActivity() {
    ......
}
```

这里的路径名**至少要有两级**，且中间以`/`分隔开。

**这三个步骤做完之后**您还需要在app主模块的`buile.gradle`中添加对所有用到ARouter模块的依赖（使用`implementation`添加依赖即可）。（Tips：这里**只需要**在主模块中添加依赖就可以了，假如要从`module_article`跳转到`module_user`，则二者之间**并不需要添加依赖**。）

然后我们就可以在`SplashActivity.kt`中编写相关的跳转逻辑了，这里为了方便我直接使用`SharedPreferences`来保存是否已登录的属性：

```kotlin
    val isLogin = getSharedPreferences("is_login", MODE_PRIVATE)
            .getBoolean("isLogin", false)
    if (isLogin){
        // 跳转到主界面
        ARouter.getInstance().build("/main/main").navigation()
    }else{
        // 跳转到登录界面
        ARouter.getInstance().build("/user/login").navigation()
    }
```

当然您也可以通过`.withXxx()`在跳转时携带参数，更多诸如添加过滤拦截器的用法可以参考[ARouter官网](https://github.com/alibaba/ARouter/blob/master/README_CN.md)。如果您在使用ARouter的过程中遇见无法跳转的问题，可以`GankApplication.kt`中的`isDebug`属性设置为`true`。

接着就是在`LoginActivity.kt`中添加登录按钮，点击后修改`SharedPreferences`的属性值并跳转到`MainActivity`，基本逻辑没什么问题，这里就不赘述，可自行查看github源码。

#### 1.5.2.2 模块/组件间数据传递

继续上面的业务逻辑，在`MainActivity`中我们需要展示用户信息和文章简介等内容，这里就以如何在`MainActivity`中获取`module_user`里`UserActivity`的用户信息为例，来说明组件间的数据传递内容。

首先，如果要通过`MainActivity`获取到`UserActivity`中的用户信息，我们需要在`module_user`中提供一个供其他模块调用的方法，例如在`module_user`中新建一个类文件`UserService.kt`，在这里提供获取用户信息的方法：

```kotlin
fun getUserId(): String? {
    return "这是用户信息"
}
```

然后在app主模块中添加对`module_user`的依赖，就可以直接实例化`UserService`对象并调用它对应的方法来获取用户信息了。但是这里存在两个问题：

1.  **模块间耦合度太高**：如果moduleA需要获取moduleB中的数据，就需要在`build.gradle`中添加对moduleB的依赖，才能使用moduleB中类的方法。
1.  **会产生循环依赖问题**：如果moduleA需要获取到moduleB中的数据，moduleB也需要获取到moduleA中的数据，那么二者就需要相互添加对彼此的依赖，这是不允许的。

其实上述的两个问题的本质还是要尽量减少模块间的耦合度。那么首先这里就有一个解决方案：将两个模块间通信的接口**下沉到底层组件lib_common中**，通过`lib_common`统一管理模块间的通信，具体的下沉逻辑请看下图：

<div align=center>
<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0f1a70b29e34564bb04e76e4169aba5~tplv-k3u1fbpfcp-watermark.image"/>
</div>

在这个图中`lib_common`起着**桥梁**的作用，对**需要传递数据的模块**（如`module_user`和`module_article`）提供相应的接口和空接口实现（实现空接口是为了防止在对应的模块未实现该接口时不会产生运行错误，而是返回默认的数据）。在`module_user`和`module_article`中分别实现对应的接口，然后在需要获取数据的模块`module_main`中调用`ServiceFactory`里相应的方法，来获得数据。具体各类的代码实现如下：

**lib_common\ServiceFactory.kt**

```kotlin
object ServiceFactory {
    // 获取用户信息的服务
    var user_service: IUserService = EmptyUserService()
    // 获取文章信息的服务
    var article_service: IArticleService = EmptyArticleService()
}
```

**lib_common\IUserService.kt**

```kotlin
interface IUserService {
    
    fun isLogin(): Boolean
    
    fun getUserId(): String?
}
```

**lib_common\EmptyUserService.kt**

```kotlin
class EmptyUserService : IUserService {
   
    override fun isLogin() = false

    override fun getUserId(): String? = null
}
```

**module_user\service\UserService.kt**

```kotlin
class UserService : IUserService {
    // 是否已登录
    override fun isLogin(): Boolean {
        return LoginActivity.isLogin
    }
    // 获取用户信息
    override fun getUserId(): String? {
        return "这是用户信息"
    }
}
```

最后我们就可以直接在`MainActivity.kt`中调用`ServiceFactory`相应的方法来获取到用户信息了：

```kotlin
ServiceFactory.user_service.getUserId()
```

当然，到这里我们获取到的数据还是空实现类中默认返回的数据，还需要在**app初始化**的时候依次创建类的实例。由于在集中调试的时候各模块的`Application`是**无法初始化**的，所以我们需要在主模块中通过**反射**依次获取各个模块的`Application`，并执行其中相应的初始化方法。

这里我们首先在`lib_common`模块中配置*所有需要进行初始化的模块*的信息，即**包名**：

**lib_common\AppConfig.kt**

```kotlin
class AppConfig {
    
    companion object{
        val moduleApps = setOf(
            "com.aefottt.module_user.LoginApp",
            "com.aefottt.module_article.ArticleApp"
        )
    }
}
```

然后新建Application的基类`BaseApp.kt`，所有模块的`Application文件`都需要继承自该`BaseApp类`：

**lib_common\BaseApp.kt**

```kotlin
abstract class BaseApp: Application() {
    
    abstract fun initModuleApp(application: Application)
}
```

在`module_user`和`module_article`中分别新建各自的`Application`文件，在其中初始化接口的实现类：

**module_user\LoginApp.kt**

```kotlin
class LoginApp : BaseApp() {
    
    override fun initModuleApp(application: Application) {
        ServiceFactory.user_service = UserService()
    }
}
```

**最后**，在app主模块的`GankApplication.kt`文件中通过反射依次获取类的实例，并调用其中的初始化方法：

```kotlin
class GankApplication: BaseApp() {

    override fun onCreate() {
        super.onCreate()

        initModuleApp(this)

        ......
    }

    override fun initModuleApp(application: Application) {
        for (appPath in AppConfig.moduleApps){
            try {
                val clazz = Class.forName(appPath)
                val baseApp = clazz.newInstance() as BaseApp
                baseApp.initModuleApp(application)
            }catch (e: Exception){
                e.printStackTrace()
            }
        }
    }

}
```

到这里通过下沉接口实现组件间的数据传递的实现就完成了。但是参考*美团Android组件化方案*，其中对于循环依赖的解决办法是**把每个业务组件都拆分成了一个Export Module和Implement Module**，其中`Export Module`复杂对外提供接口，`Implement Module`负责实现业务逻辑，不对外开发。这样的话如果`ModuleA`需要使用`ModuleB`的数据，就只需引入对`Module B的Export Module`的依赖即可，而不再需要添加对`ModuleB`的依赖了。`ModuleB`如果需要使用`ModuleA`的数据也同理，下面引入美团的一张图来说明：

<div align=center>
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7469259ae9994a83bb2f02b7aa471498~tplv-k3u1fbpfcp-watermark.image" width="68%;"/>
</div>

> 题外话：
>
> 第一次在掘金发文章，之前一直都是在看，这次借着培训时需要讲课件的机会，把我准备的课件知识分享出来，当然我写的不是最好的，还有其他人写的比我要好，然后我准备的课件主题是模块化与热修复，这一篇就是我准备的模块化的内容，关于热修复我只准备了关于类加载机制源码分析的内容，实战手撸热修复框架的过程还是遇到了困难，网上相关的文章比较久远，和现在as版本有一些出入，例如Instant Run已经被移除了，取而代之的是Change，还有就是classes.dex文件始终找不到，等着以后有时间研究好了再发出来吧。
>
> 然后这篇文章中如果有描述不足或者不正确的地方，希望大家能及时指出，谢谢您的观看。


Github链接: [Aefottt007/GankIo (github.com)](https://github.com/Aefottt007/GankIo)

参考文章：

[Android 组件化最佳实践 (juejin.cn)](https://juejin.cn/post/6844903649102004231)

[Android组件化/模块化开发(一)](https://www.jianshu.com/p/748bf621a9a0)

[android gradle 3.0.0 中依赖指令implementation、api 的区别_杨小熊学习笔记-CSDN博客](https://blog.csdn.net/Next_Second/article/details/78428086)