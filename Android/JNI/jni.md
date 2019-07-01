[TOC]

# JNI: Java Native Interface

参考书：深入理解Android卷I 第二章深入理解JNI

参考代码：<http://androidxref.com/9.0.0_r3>

参考笔记： https://juejin.im/post/5b42a9a86fb9a04f8a216b67

## 简介

###  目的：java & c/c++可以互相调用对方的函数

* 对java层屏蔽不同操作系统的差异
* native已经实现的功能可以直接用，避免重复造轮子
* native实现部分的运行效率和速度快

Java(MediaScanner) <--> JNI(libmedia**_jni**.so) <--> Native(libmedia.so)

**观察MediaScanner.java中的native方法：**

```java
public class MediaScanner
{
    static {
        System.loadLibrary("media_jni"); // 根据不同平台扩展，linux下扩展为libmedia_jni.so, windows下扩展为media_jni.dll
        native_init();
    }
    ...
    private native boolean processFile(String path, String mimeType, MediaScannerClient client);
    private static native final void native_init();
}
```

**观察android_media_MediaScanner.cpp中两个native方法的实现：**

- native_init()

```cpp
static void
android_media_MediaScanner_native_init(JNIEnv *env)
{
    ALOGV("native_init");
    jclass clazz = env->FindClass(kClassMediaScanner);
    if (clazz == NULL) {
        return;
    }

    fields.context = env->GetFieldID(clazz, "mNativeContext", "J");
    if (fields.context == NULL) {
        return;
    }
}
```

- private native void **processFile**(String path, String mimeType, MediaScannerClient client);

```cpp
static jboolean
android_media_MediaScanner_processFile(
        JNIEnv *env, jobject thiz, jstring path,
        jstring mimeType, jobject client)
{
    ALOGV("processFile");

    // Lock already hold by processDirectory
    MediaScanner *mp = getNativeScanner_l(env, thiz);
    if (mp == NULL) {
        jniThrowException(env, kRunTimeException, "No scanner available");
        return false;
    }

    if (path == NULL) {
        jniThrowException(env, kIllegalArgumentException, NULL);
        return false;
    }

    const char *pathStr = env->GetStringUTFChars(path, NULL);
    if (pathStr == NULL) {  // Out of memory
        return false;
    }

    const char *mimeTypeStr =
        (mimeType ? env->GetStringUTFChars(mimeType, NULL) : NULL);
    if (mimeType && mimeTypeStr == NULL) {  // Out of memory
        // ReleaseStringUTFChars can be called with an exception pending.
        env->ReleaseStringUTFChars(path, pathStr);
        return false;
    }

    MyMediaScannerClient myClient(env, client);
    MediaScanResult result = mp->processFile(pathStr, mimeTypeStr, myClient);
    if (result == MEDIA_SCAN_RESULT_ERROR) {
        ALOGE("An error occurred while scanning file '%s'.", pathStr);
    }
    env->ReleaseStringUTFChars(path, pathStr);
    if (mimeType) {
        env->ReleaseStringUTFChars(mimeType, mimeTypeStr);
    }
    return result != MEDIA_SCAN_RESULT_ERROR;
}
```

### java native关键字

* 该方法的实现由非java语言实现，比如C。这个特征并非java所特有，很多其它的编程语言都有这一机制，比如在C＋＋中，你可以用extern "C"告知C＋＋编译器去调用一个C的函数;
* 标识符native不能与abstract连用;
* 如果一个含有native方法的类被继承，子类会继承这个本地方法并且可以用java语言重写这个方法（这个似乎看起来有些奇怪），同样的如果一个本地方法被final标识，它被继承后不能被重写。

## 如何注册

### JNI HelloWorld实例: 静态注册

参考https://blog.csdn.net/xyang81/article/details/41777471自己实现JNI Helloworldhttps://gist.github.com/FrannyZhao/3fb01202e598495033744a40bc841d2e

静态注册是根据函数名来建立Java函数和JNI函数的对应关系的（```.对应 _ , _ 对应 _1```）.

**弊端：**

* 需要编译所有包含native方法的Java类，每个.class都需要用javah生成一个头文件；
* javah生成的JNI函数名特别长，用起来不太方便；
* 初次调用native函数时需要通过函数名字搜索对应的JNI层函数来建立关联关系，会影响效率。

### 用动态注册来解决上面的弊端

观察android_media_MediaScanner.cpp中JNINativeMethod, 标示了native函数的对应关系：

```cpp
static const JNINativeMethod gMethods[] = {
...
    {
        "processFile", // java中native函数的名字
        "(Ljava/lang/String;Ljava/lang/String;Landroid/media/MediaScannerClient;)Z", // java函数的签名信息，是参数类型和返回值类型的组合
        (void *)android_media_MediaScanner_processFile //JNI层对应函数的函数指针，注意它是void*类型
    },
...
    {
        "native_init",
        "()V",
        (void *)android_media_MediaScanner_native_init
    },
...
};
```

这个数组是这样被注册的：

调用android_media_MediaPlayer.cpp中的register_android_media_MediaScanner:

```c++
extern int register_android_media_MediaScanner(JNIEnv *env);
...
jint JNI_OnLoad(JavaVM* vm, void* /* reserved */)
{
    JNIEnv* env = NULL;
    jint result = -1;

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        ALOGE("ERROR: GetEnv failed\n");
        goto bail;
    }
    assert(env != NULL);

    if (register_android_media_ImageWriter(env) != JNI_OK) {
        ALOGE("ERROR: ImageWriter native registration failed");
        goto bail;
    }
...
    if (register_android_media_MediaScanner(env) < 0) {
        ALOGE("ERROR: MediaScanner native registration failed\n");
        goto bail;
    }
...
    if (register_android_media_MediaHTTPConnection(env) < 0) {
        ALOGE("ERROR: MediaHTTPConnection native registration failed");
        goto bail;
    }

    /* success -- return valid version number */
    result = JNI_VERSION_1_4;

bail:
    return result;
}
```


register_android_media_MediaScanner中注册JNINativeMethod数组

```cpp
int register_android_media_MediaScanner(JNIEnv *env)
{
    return AndroidRuntime::registerNativeMethods(env,
                kClassMediaScanner, gMethods, NELEM(gMethods));
    //调用AndroidRuntime的registerNativeMethods函数，第二个参数表明是java中哪个累
}
```

AndroidRuntime.cpp中的registerNativeMethods函数：

```cpp
int AndroidRuntime::registerNativeMethods(JNIEnv* env,
    const char* className, const JNINativeMethod* gMethods, int numMethods)
{
    return jniRegisterNativeMethods(env, className, gMethods, numMethods);
    //调用jniRegisterNativeMethods完成注册
}
```

JNIHelp.cpp中的jniRegisterNativeMethods函数：

```c++
extern "C" int jniRegisterNativeMethods(C_JNIEnv* env, const char* className,
    const JNINativeMethod* gMethods, int numMethods)
{
    JNIEnv* e = reinterpret_cast<JNIEnv*>(env);

    ALOGV("Registering %s's %d native methods...", className, numMethods);

    scoped_local_ref<jclass> c(env, findClass(env, className));
    if (c.get() == NULL) {
        char* tmp;
        const char* msg;
        if (asprintf(&tmp,
                     "Native registration unable to find class '%s'; aborting...",
                     className) == -1) {
            // Allocation failed, print default warning.
            msg = "Native registration unable to find class; aborting...";
        } else {
            msg = tmp;
        }
        e->FatalError(msg);
    }
//实际上是调用JNIEnv的RegisterNatives函数完成注册的
    if ((*env)->RegisterNatives(e, c.get(), gMethods, numMethods) < 0) {
        char* tmp;
        const char* msg;
        if (asprintf(&tmp, "RegisterNatives failed for '%s'; aborting...", className) == -1) {
            // Allocation failed, print default warning.
            msg = "RegisterNatives failed; aborting...";
        } else {
            msg = tmp;
        }
        e->FatalError(msg);
    }

    return 0;
}
```

### 总结：JVM加载native方法和注册流程

```flow
st=>start: JVM加载包含native方法的类
a=>operation: 该类的字节码会被加载到内存，在这个被加载的字节码的入口维持着一个该类所有方法描述符的list，
这些方法描述符包含这样一些信息：方法代码存于何处，它有哪些参数，方法的描述符（public之类）
等等;
如果一个方法描述符内有native，这个描述符块将有一个指向该方法的实现的指针
b=>operation: 通过调用java.system.loadLibrary("xxx")加载DLL或SO
bb=>start: 以下实现在加载的lib中
c=>subroutine: 查找该库中的JNI_OnLoad函数，如果有，就调用它，完成动态注册
（静态注册根据函数名完成对应关系，不需要实现这个方法）
d=>subroutine: register_android_media_MediaScanner(env)
e=>subroutine: AndroidRuntime::registerNativeMethods(env,
                kClassMediaScanner, gMethods, NELEM(gMethods));
                JNINativeMethod gMethods中定义了函数对应关系
f=>subroutine: jniRegisterNativeMethods(env, className, gMethods, numMethods);
g=>subroutine: (*env)- >FindClass(env, className);
(*env)- >RegisterNatives(env, clazz, gMethods, numMethods)
h=>inputoutput: JNI_OnLoad return JNI_VERSION_1_4 //必须返回这个值，否则会报错

st->a
a->b
b->bb
bb->c
c->d
d->e
e->f
f->g
g->h
```

## Java和JNI层数据类型转换







## JNIEnv





## 垃圾回收和异常处理







JNIEnv类型**实际上代表了Java环境，通过这个JNIEnv* 指针，就可以对Java端的代码进行操作。例如，创建Java类中的对象，调用Java对象的方法，获取Java对象中的属性等等。JNIEnv的指针会被JNI传入到本地方法的实现函数中来对Java端的代码进行操作。

JNIEnv类中有很多函数可以用：

NewObject:创建Java类中的对象

NewString:创建Java类中的String对象

New<Type>Array:创建类型为Type的数组对象

Get<Type>Field:获取类型为Type的字段

Set<Type>Field:设置类型为Type的字段的值

GetStatic<Type>Field:获取类型为Type的static的字段

SetStatic<Type>Field:设置类型为Type的static的字段的值

Call<Type>Method:调用返回类型为Type的方法

CallStatic<Type>Method:调用返回值类型为Type的static方法

等许多的函数，具体的可以查看jni.h文件中的函数名称。





