[TOC]

# JNI: Java Native Interface

### 目的：java & c/c++可以互相调用对方的函数

* 对java层屏蔽不同操作系统的差异
* native已经实现的功能可以直接用，避免重复造轮子
* native实现部分的运行效率和速度快

Java(MediaScanner) <--> JNI(libmedia**_jni**.so) <--> Native(libmedia.so)

```java
public class MediaScanner
{
    static {
        System.loadLibrary("media_jni"); // 根据不同平台扩展，linux下扩展为libmedia_jni.so, windows下扩展为media_jni.dll
        native_init();
    }
    ...
}
```

### java native关键字

* 该方法的实现由非java语言实现，比如C。这个特征并非java所特有，很多其它的编程语言都有这一机制，比如在C＋＋中，你可以用extern "C"告知C＋＋编译器去调用一个C的函数;
* 标识符native不能与abstract连用;
* 如果一个含有native方法的类被继承，子类会继承这个本地方法并且可以用java语言重写这个方法（这个似乎看起来有些奇怪），同样的如果一个本地方法被final标识，它被继承后不能被重写。

**JVM加载native方法**

```flow
st=>operation: JVM加载包含native方法的类
a=>operation: 该类的字节码会被加载到内存，在这个被加载的字节码的入口维持着一个该类所有方法描述符的list，这些方法描述符包含这样一些信息：方法代码存于何处，它有哪些参数，方法的描述符（public之类）等等
b=>operation: 如果一个方法描述符内有native，这个描述符块将有一个指向该方法的实现的指针
c=>operation: 当native方法被调用之前，这些DLL,SO才会被加载，这是通过调用java.system.loadLibrary()实现的

st->a
a->b
b->c
```

**TODO: **参考https://www.jianshu.com/p/1ba925157f7d和<https://www.jianshu.com/p/1eb6d859175d>自己实现JNI Helloworld

**观察两个native方法**

* native_init()

实现在android_media_MediaScanner.cpp中

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

* private native void **processFile**(String path, String mimeType, MediaScannerClient client);

```cpp
static void
android_media_MediaScanner_processFile(
        JNIEnv *env, jobject thiz, jstring path,
        jstring mimeType, jobject client)
{
    ALOGV("processFile");

    // Lock already hold by processDirectory
    MediaScanner *mp = getNativeScanner_l(env, thiz);
    if (mp == NULL) {
        jniThrowException(env, kRunTimeException, "No scanner available");
        return;
    }

    if (path == NULL) {
        jniThrowException(env, kIllegalArgumentException, NULL);
        return;
    }

    const char *pathStr = env->GetStringUTFChars(path, NULL);
    if (pathStr == NULL) {  // Out of memory
        return;
    }

    const char *mimeTypeStr =
        (mimeType ? env->GetStringUTFChars(mimeType, NULL) : NULL);
    if (mimeType && mimeTypeStr == NULL) {  // Out of memory
        // ReleaseStringUTFChars can be called with an exception pending.
        env->ReleaseStringUTFChars(path, pathStr);
        return;
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
}
```

**JNIEnv类型**实际上代表了Java环境，通过这个JNIEnv* 指针，就可以对Java端的代码进行操作。例如，创建Java类中的对象，调用Java对象的方法，获取Java对象中的属性等等。JNIEnv的指针会被JNI传入到本地方法的实现函数中来对Java端的代码进行操作。

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

