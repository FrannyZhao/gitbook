[toc]

# Activity

## 多窗口支持

https://developer.android.com/guide/topics/ui/multi-window

7.0开始支持分屏；

8.0开始支持画中画；

正在交互的activity处于resumed状态，可见但没有交互中的activity处于started而非resumed状态；

配置android:resizableActivity=true|false支持分屏与否；不支持多窗口模式的activity在多窗口模式下启动，将全屏显示；

配置android:supportsPictureInPicture=true|false支持画中画与否；

* API：

isInMultiWindowMode() 是否处于多窗口模式

isInPictureInPictureMode() 是否处于画中画模式

onMultiWindowModeChanged() 进入或退出多窗口模式时系统调用，进入时系统传入true

onPictureInPictureModeChanged() 进入或退出画中画模式时系统调用，进入时系统传入true

Activity.enterPictureInPictureMode() 将activity置于画中画模式（若设备不支持画中画模式则此方法无效）

### Todo

1. 

```xml
<activity android:name=".MyActivity">
    <layout android:defaultHeight="500dp"
          android:defaultWidth="600dp"
          android:gravity="top|end"
          android:minHeight="450dp"
          android:minWidth="300dp" />
</activity>
```

2. 拖放

3. 使用 Intent 标志 `FLAG_ACTIVITY_LAUNCH_ADJACENT`。此标志将告知系统尽量在启动它的 Activity 旁边创建新 Activity，以便两个 Activity 共享屏幕。系统会尽可能这些做，但最终结果无法保证。
4. 如果设备处于自由窗口模式，则在启动新 Activity 时，您可通过调用 `ActivityOptions.setLaunchBounds()` 来指定新 Activity 的尺寸和屏幕位置。如果设备未处于多窗口模式，则调用该方法不会产生任何影响。



## 可折叠设备的适配

在搭载 Android 9.0 及更低版本的设备上运行时，只有获得焦点的应用处于已恢复状态。任何其他可见 Activity 都处于已暂停状态。如果应用在处于暂停状态时关闭资源或停止播放内容，则可能会产生问题。

在 Android 10 中，此行为发生了变化，即当设备处于多窗口模式时，所有 Activity 都会保持已恢复状态。这称为*多项恢复*。请注意，如果顶部有透明 Activity，或者 Activity 不可成为焦点（例如，处于画中画模式），则相应 Activity 可能会处于已暂停状态。还有一种可能是，在特定时间内（例如，当打开抽屉式通知栏时）所有 Activity 都不具有焦点。`OnStop` 会继续照常工作。每当 Activity 从屏幕上移除时，系统都会调用它.



* 新生命周期回调

```java
protected void onTopResumedActivityChanged(boolean topResumed) {
    if (topResumed) {
        // Top resumed activity
        // Can be a signal to re-acquire exclusive resources
    } else {
        // No longer the top resumed activity
    }
}
```

当 Activity 使用共享的单用户资源（例如麦克风或摄像头）时，了解这一点至关重要.

对于使用摄像头的应用，建议使用 `CameraManager.AvailabilityCallback#onCameraAccessPrioritiesChanged()`方法作为提示，提醒这可能是尝试访问摄像头的好时机。

应用收到 `CameraDevice.StateCallback#onDisconnected()` 回调后，对摄像头设备进行的后续调用将抛出 `CameraAccessException`。



# 架构

## App startup

需要启动的模块：

1. 实现Initializer<T>
2. 重写create和dependencies方法

```java
// Initializes ExampleLogger.
class ExampleLoggerInitializer implements Initializer<ExampleLogger> {

    @Override
    public ExampleLogger create(Context context) {
        // WorkManager.getInstance() is non-null only after
        // WorkManager is initialized.
        return ExampleLogger(WorkManager.getInstance(context));
    }

    @Override
    public List<Class<Initializer<?>>> dependencies() {
        // Defines a dependency on WorkManagerInitializer so it can be
        // initialized after WorkManager is initialized.
        return Arrays.asList(WorkManagerInitializer.class);
    }
}
```



启动这个模块有两种方式：在manifest中配置到`androidx.startup.InitializationProvider`中；或者调用AppInitializer启动。

1. 在manifest中配置.

```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes ExampleLoggerInitializer discoverable. -->
    <meta-data  android:name="com.example.ExampleLoggerInitializer"
          android:value="androidx.startup" />
</provider>
```

或者

2. 调用AppInitializer

```java
AppInitializer.getInstance(context)
    .initializeComponent(ExampleLoggerInitializer.class);
```



## Data Store

SharedPreference缺点：不支持跨进程；sp文件数据会全部存在内存中，不宜存放大数据；任何修改都是全量写入，建议对高频修改的数据放在单独的sp文件中；

如果不需要跨进程，仅存储少量配置项，那么可以选择SharedPreference.

DataStore解决了这些缺点，可以保证原子性，一致性，隔离性，持久性，线程安全，非阻塞。是SP的优秀替代。

但不支持局部更新，也不保证引用完整性，如果需要这些则应选择用Room。

对比：

| DataStore              | Room                                                         |
| ---------------------- | ------------------------------------------------------------ |
| small, simple datasets | large, complex datasets;<br/>partial updates;<br/>referential integrity(引用完整性) |

引用完整性：要求关系中不允许引用不存在的实体。**whenever a foreign key value is used it must reference a valid, existing primary key in the parent table.**

Preference Datastore: 键值对存储，不要求预定模版，不要求类型安全；

Proto DataStore: 存储类的对象，通过protocol buffers将对象序列化存储，要求类型安全。

### Todo

学了kotlin后实践



## 视图绑定

https://developer.android.com/topic/libraries/view-binding#java

Fragment 的存在时间比其视图长。请务必在 Fragment 的 `onDestroyView()` 方法中清除对绑定类实例的所有引用。

```java
    private ResultProfileBinding binding;

    @Override
    public View onCreateView (LayoutInflater inflater,
                              ViewGroup container,
                              Bundle savedInstanceState) {
        binding = ResultProfileBinding.inflate(inflater, container, false);
        View view = binding.getRoot();
        return view;
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        binding = null;
    }
    
```

与使用 `findViewById` 相比，视图绑定具有一些很显著的优点：

- **Null 安全**：由于视图绑定会创建对视图的直接引用，因此不存在因视图 ID 无效而引发 Null 指针异常的风险。此外，如果视图仅出现在布局的某些配置中，则绑定类中包含其引用的字段会使用 `@Nullable` 标记。
- **类型安全**：每个绑定类中的字段均具有与它们在 XML 文件中引用的视图相匹配的类型。这意味着不存在发生类转换异常的风险。

这些差异意味着布局和代码之间的不兼容将会导致构建在编译时（而非运行时）失败。



## 数据绑定

* Null 合并运算符

如果左边运算数不是 `null`，则 Null 合并运算符 (`??`) 选择左边运算数，如果左边运算数为 `null`，则选择右边运算数。

```java
android:text="@{user.displayName ?? user.lastName}"
```

等于

```java
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

* 视图引用

绑定类将 ID 转换为驼峰式大小写。

```xml
<EditText
        android:id="@+id/example_text"
.../>
    <TextView
        android:id="@+id/example_output"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{exampleText.text}"/>
```













































