



[TOC]

# Android强制更改为横屏方案

## 需求

显示屏的长宽是在kernel里面定义的，甲方给的android sdk里面没有kernel的代码，无法修改，只能在aosp里面修改代码，使系统从竖屏变成横屏的。

参考https://blog.csdn.net/songjinshi/article/details/50586333

* 增加了应用层的修改，防止有的应用写了screenOrientation = portrait, 或者request orientation portrait
* 增加了recovery显示的旋转

## frameworks/native

### SurfaceFlinger的DispalyDevice

```cpp
diff --git a/services/surfaceflinger/DisplayDevice.cpp b/services/surfaceflinger/DisplayDevice.cpp
index 6128890..3d92e49 100644
--- a/services/surfaceflinger/DisplayDevice.cpp
+++ b/services/surfaceflinger/DisplayDevice.cpp
@@ -189,7 +189,8 @@ DisplayDevice::DisplayDevice(
     }
 
     // initialize the display orientation transform.
-    setProjection(DisplayState::eOrientationDefault, mViewport, mFrame);
+    //setProjection(DisplayState::eOrientationDefault, mViewport, mFrame);
+    setProjection(DisplayState::eOrientation90, mViewport, mFrame);
 
     if (useTripleFramebuffer) {
         surface->allocateBuffers();
@@ -563,6 +564,10 @@ void DisplayDevice::setProjection(int orientation,
         // the destination frame can be invalid if it has never been set,
         // in that case we assume the whole display frame.
         frame = Rect(w, h);
+        if (R.getOrientation() & Transform::ROT_90) {
+            // for force landscape
+            swap(frame.right, frame.bottom);
+        }
     }
 
     if (viewport.isEmpty()) {

```



## frameworks/base

### 开机动画，宽高对调

```cpp
diff --git a/cmds/bootanimation/BootAnimation.cpp b/cmds/bootanimation/BootAnimation.cpp
index 6526123..abb8510 100644
--- a/cmds/bootanimation/BootAnimation.cpp
+++ b/cmds/bootanimation/BootAnimation.cpp
@@ -258,7 +258,8 @@ status_t BootAnimation::readyToRun() {
 
     // create the native surface
     sp<SurfaceControl> control = session()->createSurface(String8("BootAnimation"),
-            dinfo.w, dinfo.h, PIXEL_FORMAT_RGB_565);
+            dinfo.h, dinfo.w, PIXEL_FORMAT_RGB_565);
+            //dinfo.w, dinfo.h, PIXEL_FORMAT_RGB_565);
 
     SurfaceComposerClient::openGlobalTransaction();
     control->setLayer(0x40000000);

```

### framework层


```java
diff --git a/services/core/java/com/android/server/policy/PhoneWindowManager.java b/services/core/java/com/android/server/policy/PhoneWindowManager.java
index e81fd66..bfd068a 100644
--- a/services/core/java/com/android/server/policy/PhoneWindowManager.java
+++ b/services/core/java/com/android/server/policy/PhoneWindowManager.java
@@ -7167,7 +7167,9 @@ public class PhoneWindowManager implements WindowManagerPolicy {
                             ? "USER_ROTATION_LOCKED" : "")
                         );
         }
-
+        if (true) {
+            return mLandscapeRotation;
+        }
         if (mForceDefaultOrientation) {
             return Surface.ROTATION_0;
         }

```

```java
diff --git a/services/core/java/com/android/server/wm/DisplayContent.java b/services/core/java/com/android/server/wm/DisplayContent.java
index a49f447..be35409 100644
--- a/services/core/java/com/android/server/wm/DisplayContent.java
+++ b/services/core/java/com/android/server/wm/DisplayContent.java
@@ -233,7 +233,7 @@ class DisplayContent extends WindowContainer<DisplayContent.DisplayChildWindowCo
      *
      * @see #updateRotationUnchecked(boolean)
      */
-    private int mRotation = 0;
+    private int mRotation = 1;
 
     /**
      * Last applied orientation of the display.
@@ -891,7 +891,8 @@ class DisplayContent extends WindowContainer<DisplayContent.DisplayChildWindowCo
     }
 
     void setRotation(int newRotation) {
-        mRotation = newRotation;
+        //mRotation = newRotation;
+        mRotation = 1;
     }
 
     int getLastOrientation() {

```

### 应用层

```java
diff --git a/core/java/android/app/Activity.java b/core/java/android/app/Activity.java
index bfa35da..e0c53eb 100644
--- a/core/java/android/app/Activity.java
+++ b/core/java/android/app/Activity.java
@@ -5773,12 +5773,12 @@ public class Activity extends ContextThemeWrapper
         if (mParent == null) {
             try {
                 ActivityManager.getService().setRequestedOrientation(
-                        mToken, requestedOrientation);
+                        mToken, 0);
             } catch (RemoteException e) {
                 // Empty
             }
         } else {
-            mParent.setRequestedOrientation(requestedOrientation);
+            mParent.setRequestedOrientation(0);
         }
     }
 
```

```java
diff --git a/core/java/android/content/pm/PackageParser.java b/core/java/android/content/pm/PackageParser.java
index 750bad6..6b34af6 100644
--- a/core/java/android/content/pm/PackageParser.java
+++ b/core/java/android/content/pm/PackageParser.java
@@ -4378,9 +4378,7 @@ public class PackageParser {
                 a.info.flags |= ActivityInfo.FLAG_RESUME_WHILE_PAUSING;
             }
 
-            a.info.screenOrientation = sa.getInt(
-                    R.styleable.AndroidManifestActivity_screenOrientation,
-                    SCREEN_ORIENTATION_UNSPECIFIED);
+            a.info.screenOrientation = 0;
 
             setActivityResizeMode(a.info, sa, owner);
 
@@ -4810,7 +4808,7 @@ public class PackageParser {
         if (info.descriptionRes == 0) {
             info.descriptionRes = target.info.descriptionRes;
         }
-        info.screenOrientation = target.info.screenOrientation;
+        info.screenOrientation = 0;
         info.taskAffinity = target.info.taskAffinity;
         info.theme = target.info.theme;
         info.softInputMode = target.info.softInputMode;
```

## bootable/recovery

### recovery UI

用命令`convert progress_fill.png -rotate 90 progress_fill.png && convert progress_empty.png -rotate 90 progress_empty.png` 把以下图片旋转90度：

res-hdpi/images/progress_empty.png
 res-hdpi/images/progress_fill.png
 res-mdpi/images/progress_empty.png
 res-mdpi/images/progress_fill.png
 res-xhdpi/images/progress_empty.png
 res-xhdpi/images/progress_fill.png
 res-xxhdpi/images/progress_empty.png
 res-xxhdpi/images/progress_fill.png
 res-xxxhdpi/images/progress_empty.png
 res-xxxhdpi/images/progress_fill.png

```cpp
diff --git a/screen_ui.cpp b/screen_ui.cpp
index b8f6ea2..153d1b1 100644
--- a/screen_ui.cpp
+++ b/screen_ui.cpp
@@ -170,8 +170,8 @@ void ScreenRecoveryUI::draw_background_locked() {
     }
 
     GRSurface* text_surface = GetCurrentText();
-    int text_x = (gr_fb_width() - gr_get_width(text_surface)) / 2;
-    int text_y = GetTextBaseline();
+    int text_x = (gr_fb_width() - gr_get_width(text_surface)) / 3; //(gr_fb_width() - gr_get_width(text_surface)) / 2;
+    int text_y = (gr_fb_height() - gr_get_height(text_surface)) / 2; //GetTextBaseline();
     gr_color(255, 255, 255, 255);
     gr_texticon(text_x, text_y, text_surface);
   }
@@ -184,8 +184,8 @@ void ScreenRecoveryUI::draw_foreground_locked() {
     GRSurface* frame = GetCurrentFrame();
     int frame_width = gr_get_width(frame);
     int frame_height = gr_get_height(frame);
-    int frame_x = (gr_fb_width() - frame_width) / 2;
-    int frame_y = GetAnimationBaseline();
+    int frame_x = (gr_fb_width() - frame_width) * 2 / 3; //(gr_fb_width() - frame_width) / 2;
+    int frame_y = (gr_fb_height() - frame_height) / 2; //GetAnimationBaseline();
     gr_blit(frame, 0, 0, frame_width, frame_height, frame_x, frame_y);
   }
 
@@ -193,8 +193,8 @@ void ScreenRecoveryUI::draw_foreground_locked() {
     int width = gr_get_width(progressBarEmpty);
     int height = gr_get_height(progressBarEmpty);
 
-    int progress_x = (gr_fb_width() - width) / 2;
-    int progress_y = GetProgressBaseline();
+    int progress_x = (gr_fb_width() - width) / 5; //(gr_fb_width() - width) / 2;
+    int progress_y = (gr_fb_height() - height) / 2; //GetProgressBaseline();
 
     // Erase behind the progress bar (in case this was a progress-only update)
     gr_color(0, 0, 0, 255);
@@ -202,24 +202,27 @@ void ScreenRecoveryUI::draw_foreground_locked() {
 
     if (progressBarType == DETERMINATE) {
       float p = progressScopeStart + progress * progressScopeSize;
-      int pos = static_cast<int>(p * width);
+      int pos = static_cast<int>(p * height);
 
       if (rtl_locale_) {
         // Fill the progress bar from right to left.
         if (pos > 0) {
-          gr_blit(progressBarFill, width - pos, 0, pos, height, progress_x + width - pos,
-                  progress_y);
+          //gr_blit(progressBarFill, width - pos, 0, pos, height, progress_x + width - pos, progress_y);
+          gr_blit(progressBarFill, 0, height - pos, width, pos, progress_x, progress_y + width - pos);
         }
-        if (pos < width - 1) {
-          gr_blit(progressBarEmpty, 0, 0, width - pos, height, progress_x, progress_y);
+        if (pos < height - 1) {
+          //gr_blit(progressBarEmpty, 0, 0, width - pos, height, progress_x, progress_y);
+          gr_blit(progressBarEmpty, 0, 0, width, height - pos, progress_x, progress_y);
         }
       } else {
         // Fill the progress bar from left to right.
         if (pos > 0) {
-          gr_blit(progressBarFill, 0, 0, pos, height, progress_x, progress_y);
+          //gr_blit(progressBarFill, 0, 0, pos, height, progress_x, progress_y);
+          gr_blit(progressBarFill, 0, 0, width, pos, progress_x, progress_y);
         }
-        if (pos < width - 1) {
-          gr_blit(progressBarEmpty, pos, 0, width - pos, height, progress_x + pos, progress_y);
+        if (pos < height - 1) {
+          //gr_blit(progressBarEmpty, pos, 0, width - pos, height, progress_x + pos, progress_y);
+          gr_blit(progressBarEmpty, 0, pos, width, height - pos, progress_x, progress_y + pos);
         }
       }
     }
@@ -460,6 +463,7 @@ void ScreenRecoveryUI::LoadBitmap(const char* filename, GRSurface** surface) {
 
 void ScreenRecoveryUI::LoadLocalizedBitmap(const char* filename, GRSurface** surface) {
   int result = res_create_localized_alpha_surface(filename, locale_.c_str(), surface);
+  RotateSurface(*surface);
   if (result < 0) {
     LOG(ERROR) << "couldn't load bitmap " << filename << " (error " << result << ")";
   }
@@ -611,8 +615,8 @@ void ScreenRecoveryUI::SetProgress(float fraction) {
   if (fraction > 1.0) fraction = 1.0;
   if (progressBarType == DETERMINATE && fraction > progress) {
     // Skip updates that aren't visibly different.
-    int width = gr_get_width(progressBarEmpty);
-    float scale = width * progressScopeSize;
+    int height = gr_get_height(progressBarEmpty);
+    float scale = height * progressScopeSize;
     if ((int)(progress * scale) != (int)(fraction * scale)) {
       progress = fraction;
       update_progress_locked();
@@ -841,3 +845,29 @@ void ScreenRecoveryUI::KeyLongPress(int) {
   // will change color to indicate a successful long press.
   Redraw();
 }
+
+void ScreenRecoveryUI::RotateSurface(GRSurface *surface) {
+  if (surface == NULL) return;
+  if (surface->pixel_bytes != 1) return;
+  int data_size = surface->height * surface->row_bytes;
+  unsigned char*  src = new unsigned char[data_size];
+  if (src == NULL) return;
+  memcpy(src, surface->data, data_size);
+  unsigned char *dst = surface->data;
+  int width = surface->width, height = surface->height;
+  for (int j = 0; j < height; j++) {
+    unsigned char *sx = src + j * surface->row_bytes;
+    unsigned char *px = dst + (height - 1 - j) * surface->pixel_bytes;
+    for (int i = 0; i < width; i++) {
+      *px = *sx;
+      sx++;
+      px += height;
+    }
+  }
+  surface->width = height;
+  surface->height = width;
+  surface->row_bytes = surface->pixel_bytes * surface->width;
+  delete []src;
+  return;
+}
+   
```

```cpp
diff --git a/screen_ui.h b/screen_ui.h
index 8231a2b..6ccd232 100644
--- a/screen_ui.h
+++ b/screen_ui.h
@@ -191,6 +191,9 @@ class ScreenRecoveryUI : public RecoveryUI {
   // Similar to DrawTextLines() to draw multiple text lines, but additionally wraps long lines.
   // Returns the offset it should be moving along Y-axis.
   int DrawWrappedTextLines(int x, int y, const char* const* lines) const;
+
+  // Rotate 90 degree
+  void RotateSurface(GRSurface *);
 };
 
 #endif  // RECOVERY_UI_H
```

