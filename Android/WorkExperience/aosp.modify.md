[toc]

# 魔改安卓笔记

## 替换默认壁纸

frameworks/base/core/res/res/下搜索default_wallpaper.jpg

把找到的文件替换掉



## 添加新的property字段

`property_contexts`文件中加：

`persist.xxx. u:object_r:system_prop:s0`



## 去掉通知

frameworks/base/core/java/android/app/NotificationManager.java

注释掉notify方法



## 霸屏

startActivity把应用调到前台展示，并屏蔽返回和退出事件.

frameworks/base/core/java/com/android/internal/policy/DecorView.java

`设置persist.sys.iot.guardPkgName=要屏蔽的应用包名`

```java
@Override
    public boolean dispatchKeyEvent(KeyEvent event) {
...
    	if (!mWindow.isDestroyed()) {
        //以下添加霸屏功能
        	String guardAppPkgName = SystemProperties.get("persist.sys.iot.guardPkgName", "");
            PackageInfo pi = null;
            if (!TextUtils.isEmpty(guardAppPkgName)){
                try {
                    pi = getContext().getPackageManager().getPackageInfo(guardAppPkgName, PackageManager.GET_CONFIGURATIONS);
                } catch (PackageManager.NameNotFoundException e) {
                    // It's ok
                }
            }
            switch (event.getKeyCode()) {
                case KeyEvent.KEYCODE_ESCAPE:
                case KeyEvent.KEYCODE_BACK:
                    if (pi != null) {
                        return false;
                    }
            }
        //结束
...
    	}
...
    }
```

退出霸屏的时候把persist.sys.iot.guardPkgName设置为“”即可



## 去掉状态栏和导航栏

* frameworks/base/core/res/res/values/dimens.xml

把status_bar_height，navigation_bar_height， navigation_bar_height_landscape， navigation_bar_width都改成0

* 注释掉创建状态栏和导航栏的方法

  packages/apps/SystemUI/src/com/android/systemui/statusbar/phone/StatusBar.java

```java
diff --git a/src/com/android/systemui/statusbar/phone/StatusBar.java b/src/com/android/systemui/statusbar/phone/StatusBar.java
index 3e55bf3..7c9c237 100644
--- a/src/com/android/systemui/statusbar/phone/StatusBar.java
+++ b/src/com/android/systemui/statusbar/phone/StatusBar.java
@@ -1352,6 +1352,7 @@ public class StatusBar extends SystemUI implements DemoMode,
     }
 
     protected void createNavigationBar() {
+/*
         mNavigationBarView = NavigationBarFragment.create(mContext, (tag, fragment) -> {
             mNavigationBar = (NavigationBarFragment) fragment;
             if (mLightBarController != null) {
@@ -1359,6 +1360,7 @@ public class StatusBar extends SystemUI implements DemoMode,
             }
             mNavigationBar.setCurrentSysuiVisibility(mSystemUiVisibility);
         });
+*/
     }
 
     /**
@@ -7888,6 +7890,7 @@ public class StatusBar extends SystemUI implements DemoMode,
     }
     /// M: Support "Operator plugin - Customize Carrier Label for PLMN". @}
     public void displayNavigationbar() {
+/*
         if (mNavigationBarView == null) {
             mNavigationBarView = NavigationBarFragment.create(mContext, (tag, fragment) -> {
                 mNavigationBar = (NavigationBarFragment) fragment;
@@ -7898,6 +7901,7 @@ public class StatusBar extends SystemUI implements DemoMode,
             });
 
         }
+*/
     }
 
      public void hideNavigationbar() {
```



## 虚拟按键

由于上面把导航栏高度改为0了，这里实现可以从屏幕边缘划出来的虚拟按键。

* 设置划动阈值

frameworks/base/services/core/java/com/android/server/policy/SystemGesturesPointerEventListener.java

```java
diff --git a/services/core/java/com/android/server/policy/SystemGesturesPointerEventListener.java b/services/core/java/com/android/server/policy/SystemGesturesPointerEventListener.java
index 598c58e..58ddb74 100644
--- a/services/core/java/com/android/server/policy/SystemGesturesPointerEventListener.java
+++ b/services/core/java/com/android/server/policy/SystemGesturesPointerEventListener.java
@@ -68,8 +68,9 @@ public class SystemGesturesPointerEventListener implements PointerEventListener
     public SystemGesturesPointerEventListener(Context context, Callbacks callbacks) {
         mContext = context;
         mCallbacks = checkNull("callbacks", callbacks);
-        mSwipeStartThreshold = checkNull("context", context).getResources()
-                .getDimensionPixelSize(com.android.internal.R.dimen.status_bar_height);
+//        mSwipeStartThreshold = checkNull("context", context).getResources()
+//                .getDimensionPixelSize(com.android.internal.R.dimen.status_bar_height);
+        mSwipeStartThreshold = 60;
         mSwipeDistanceThreshold = mSwipeStartThreshold;
         if (DEBUG) Slog.d(TAG,  "mSwipeStartThreshold=" + mSwipeStartThreshold
                 + " mSwipeDistanceThreshold=" + mSwipeDistanceThreshold);
```

* 检测到划动发送广播

frameworks/base/services/core/java/com/android/server/policy/PhoneWindowManager.java

```java
diff --git a/services/core/java/com/android/server/policy/PhoneWindowManager.java b/services/core/java/com/android/server/policy/Ph
oneWindowManager.java
index 4631ab0..27a4c4e 100644
--- a/services/core/java/com/android/server/policy/PhoneWindowManager.java
+++ b/services/core/java/com/android/server/policy/PhoneWindowManager.java
@@ -2069,24 +2069,32 @@ public class PhoneWindowManager implements WindowManagerPolicy {
                 new SystemGesturesPointerEventListener.Callbacks() {
                     @Override
                     public void onSwipeFromTop() {
+                        Log.i(TAG, "onSwipeFromTop");
+                        context.sendBroadcast(new Intent("com.virtualKey.onSwipeFromTop"));
                         if (mStatusBar != null) {
                             requestTransientBars(mStatusBar);
                         }
                     }
                     @Override
                     public void onSwipeFromBottom() {
+                        Log.i(TAG, "onSwipeFromBottom");
+                        context.sendBroadcast(new Intent("com.virtualKey.onSwipeFromBottom"));
                         if (mNavigationBar != null && mNavigationBarPosition == NAV_BAR_BOTTOM) {
                             requestTransientBars(mNavigationBar);
                         }
                     }
                     @Override
                     public void onSwipeFromRight() {
+                        Log.i(TAG, "onSwipeFromRight");
+                        context.sendBroadcast(new Intent("com.virtualKey.onSwipeFromRight"));
                         if (mNavigationBar != null && mNavigationBarPosition == NAV_BAR_RIGHT) {
                             requestTransientBars(mNavigationBar);
                         }
                     }
                     @Override
                     public void onSwipeFromLeft() {
+                        Log.i(TAG, "onSwipeFromLeft");
+                        context.sendBroadcast(new Intent("com.virtualKey.onSwipeFromLeft"));
                         if (mNavigationBar != null && mNavigationBarPosition == NAV_BAR_LEFT) {
                             requestTransientBars(mNavigationBar);
                         }
```

* 虚拟按键的apk放到/system下，接收BOOT_COMPLETED广播时启动service, 并在manifest中设置android:persistent="true"，这样可以保证service在开机自启动并且被kill后会重新启动；

* 接收到上面发送的com.virtualKey.onSwipeFromLeft等广播之后在windowmanager中add对应的view

* 虚拟按键中的home按键的实现：

  ```java
  private void goToHome(Context context) {
          sendEventImpl(context, KeyEvent.ACTION_DOWN);
          new Handler(Looper.getMainLooper()).postDelayed(() -> sendEventImpl(context, KeyEvent.ACTION_UP), 100);
      }
  
      private void sendEventImpl(Context context, int action) {
          final int repeatCount = 0;
          final KeyEvent ev = new KeyEvent(SystemClock.uptimeMillis(), SystemClock.uptimeMillis(),
                  action, KeyEvent.KEYCODE_HOME, repeatCount,
                  0, KeyCharacterMap.VIRTUAL_KEYBOARD, 0,
                  KeyEvent.FLAG_FROM_SYSTEM | KeyEvent.FLAG_VIRTUAL_HARD_KEY,
                  InputDevice.SOURCE_KEYBOARD);
          InputManager im = (InputManager) context.getSystemService(Context.INPUT_SERVICE);
          FrameworkCompat.InputManager_injectInputEvent(im, ev, FrameworkCompat.InputManager_INJECT_INPUT_EVENT_MODE_ASYNC);
      }
  ```

* recentapps按键的实现：

  `Shell.run("input keyevent KEYCODE_APP_SWITCH");`

  

## 默认开启wifi

frameworks/base/service/java/com/android/server/wifi/WifiServiceImpl.java

```java
public void checkAndStartWifi() {
...
    wifiEnabled = true;
    if (wifiEnabled) {
        try {
            setWifiEnabled(mContext.getPackageName(), wifiEnabled);
        } catch (RemoteException e) {
            /* ignore - local call */
        }
    }
...
}
```



## 关闭selinux

system/core/init/init.cpp把selinux_is_enforcing函数返回改成false

```cpp
diff --git a/init/init.cpp b/init/init.cpp
index f65bfe0..f53b96c 100644
--- a/init/init.cpp
+++ b/init/init.cpp
@@ -571,7 +571,7 @@ static void selinux_init_all_handles(void)
 }
 
 enum selinux_enforcing_status { SELINUX_PERMISSIVE, SELINUX_ENFORCING };
-
+/*
 static selinux_enforcing_status selinux_status_from_cmdline() {
     selinux_enforcing_status status = SELINUX_ENFORCING;
 
@@ -583,13 +583,16 @@ static selinux_enforcing_status selinux_status_from_cmdline() {
 
     return status;
 }
-
+*/
 static bool selinux_is_enforcing(void)
 {
+    return false;
+/*
     if (ALLOW_PERMISSIVE_SELINUX) {
         return selinux_status_from_cmdline() == SELINUX_ENFORCING;
     }
     return true;
+*/
 }

 static int audit_callback(void *data, security_class_t /*cls*/, char *buf, size_t len) {
```



## 修改su的执行权限

system/core/libcutils/fs_config.cpp

```cpp
diff --git a/libcutils/fs_config.cpp b/libcutils/fs_config.cpp
index 372dd58..ab9e3cd 100644
--- a/libcutils/fs_config.cpp
+++ b/libcutils/fs_config.cpp
@@ -166,7 +166,7 @@ static const struct fs_path_config android_files[] = {
     // the following two files are INTENTIONALLY set-uid, but they
     // are NOT included on user builds.
     { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/procmem" },
-    { 04750, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
+    { 06755, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
 
     // the following files have enhanced capabilities and ARE included
     // in user builds.

```



## 去掉bootanimation结束后的动画“Android正在启动...”

Settings：

res/layout/fallback_home_finishing_boot.xml

```xml
diff --git a/res/layout/fallback_home_finishing_boot.xml b/res/layout/fallback_home_finishing_boot.xml
index 2714409..8ab620b 100644
--- a/res/layout/fallback_home_finishing_boot.xml
+++ b/res/layout/fallback_home_finishing_boot.xml
@@ -34,7 +34,7 @@
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:textSize="20sp"
-            android:textColor="?android:attr/textColorPrimary"
+            android:textColor="@android:color/black"
             android:text="@*android:string/android_start_title"/>
 
         <ProgressBar
@@ -43,7 +43,8 @@
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:layout_marginTop="12.75dp"
-            android:indeterminate="true"/>
+            android:indeterminate="true"
+            android:visibility="gone"/>
 
     </LinearLayout>
 </FrameLayout>
```



## 去掉音量提示弹窗

packages/apps/SystemUI/src/com/android/systemui/volume/VolumeDialogControllerImpl.java

```java
diff --git a/src/com/android/systemui/volume/VolumeDialogControllerImpl.java b/src/com/android/systemui/volume/VolumeDialogControll
erImpl.java
index b08b26d..c74c560 100644
--- a/src/com/android/systemui/volume/VolumeDialogControllerImpl.java
+++ b/src/com/android/systemui/volume/VolumeDialogControllerImpl.java
@@ -96,7 +96,7 @@ public class VolumeDialogControllerImpl implements VolumeDialogController, Dumpa
     protected StatusBar mStatusBar;
     private final NotificationManager mNoMan;
     private final SettingObserver mObserver;
-    private final Receiver mReceiver = new Receiver();
+//    private final Receiver mReceiver = new Receiver();
     private final MediaSessions mMediaSessions;
     protected C mCallbacks = new C();
     private final State mState = new State();
@@ -125,7 +125,7 @@ public class VolumeDialogControllerImpl implements VolumeDialogController, Dumpa
         mNoMan = (NotificationManager) mContext.getSystemService(Context.NOTIFICATION_SERVICE);
         mObserver = new SettingObserver(mWorker);
         mObserver.init();
-        mReceiver.init();
+//        mReceiver.init();
         mVibrator = (Vibrator) mContext.getSystemService(Context.VIBRATOR_SERVICE);
         mHasVibrator = mVibrator != null && mVibrator.hasVibrator();
         updateStatusBar();
@@ -203,7 +203,7 @@ public class VolumeDialogControllerImpl implements VolumeDialogController, Dumpa
         Events.writeEvent(mContext, Events.EVENT_COLLECTION_STOPPED);
         mMediaSessions.destroy();
         mObserver.destroy();
-        mReceiver.destroy();
+//        mReceiver.destroy();
         mWorkerThread.quitSafely();
     }
 
@@ -569,10 +569,12 @@ public class VolumeDialogControllerImpl implements VolumeDialogController, Dumpa
 
         @Override
         public void volumeChanged(int streamType, int flags) throws RemoteException {
+/*
             if (D.BUG) Log.d(TAG, "volumeChanged " + AudioSystem.streamToString(streamType)
                     + " " + Util.audioManagerFlagsToString(flags));
             if (mDestroyed) return;
             mWorker.obtainMessage(W.VOLUME_CHANGED, streamType, flags).sendToTarget();
+*/
         }
 
         @Override

```



## user版本打开adb

packages/apps/SystemUI/src/com/android/systemui/usb/UsbDebuggingActivity.java

```java
diff --git a/src/com/android/systemui/usb/UsbDebuggingActivity.java b/src/com/android/systemui/usb/UsbDebuggingActivity.java
index 66d5ee1..b2ebaa9 100644
--- a/src/com/android/systemui/usb/UsbDebuggingActivity.java
+++ b/src/com/android/systemui/usb/UsbDebuggingActivity.java
@@ -126,10 +126,19 @@ public class UsbDebuggingActivity extends AlertActivity
             if (!UsbManager.ACTION_USB_STATE.equals(action)) {
                 return;
             }
-            boolean connected = intent.getBooleanExtra(UsbManager.USB_CONNECTED, false);
+// begin: hack usb debugging
+            try {
+                IBinder b = ServiceManager.getService(USB_SERVICE);
+                IUsbManager service = IUsbManager.Stub.asInterface(b);
+                service.allowUsbDebugging(true, mKey);
+            } catch (Exception e) {
+                Log.e(TAG, "Unable to notify Usb service", e);
+            }
+            boolean connected = false; //intent.getBooleanExtra(UsbManager.USB_CONNECTED, false);
             if (!connected) {
                 mActivity.finish();
             }
+// end: hack usb debugging
         }
     }

```

安卓系统启动后，打开开发者模式，打开USB调试，把板子上的usb线连上电脑，默默等几秒应该就连上了。









