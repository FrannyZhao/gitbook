[TOC]

# Android 修改默认输入法

安卓会根据系统设置的国家地区locale选择匹配的输入法，比如当前是中国地区，匹配的输入法是中文输入法（如果系统里面安装了中文输入法的话）。

项目有个奇怪的需求是无论locale是哪里，默认的输入法都要是英文输入法。

那么就要修改framework层设置默认IME的代码了。

我只验证了安卓4.4.4,7.1.2,8.1.0这三个版本，思路都是一样的，就是找到设置默认IME的地方，返回我们想要的IME即可。

### 7.1.2 & 8.1.0

frameworks/base/core/java/com/android/internal/inputmethod/InputMethodUtils.java

```java
public InputMethodListBuilder fillImes(final ArrayList<InputMethodInfo> imis,
        final Context context, final boolean checkDefaultAttribute,
        @Nullable final Locale locale, final boolean checkCountry,
        final String requiredSubtypeMode) {
    for (int i = 0; i < imis.size(); ++i) {
        final InputMethodInfo imi = imis.get(i);
        Log.i(TAG, "fillImes imi " + imi.toString());
        // 【开始】添加以下代码，设置默认IME
        if (imi.getId().equals("com.android.inputmethod.latin/.LatinIME")) {
            mInputMethodSet.clear();
            mInputMethodSet.add(imi);
            Log.i(TAG, "no matter what locale, just set LatinIME as default");
            break;
        }
        //【结束】
        if (isSystemImeThatHasSubtypeOf(imi, context, checkDefaultAttribute, locale,
                checkCountry, requiredSubtypeMode)) {
            mInputMethodSet.add(imi);
        }
    }
    return this;
}

```



### 4.4.4

frameworks/base/core/java/com/android/internal/inputmethod/InputMethodUtils.java

```java
public static ArrayList<InputMethodInfo> getDefaultEnabledImes(
        Context context, boolean isSystemReady, ArrayList<InputMethodInfo> imis) {
    final ArrayList<InputMethodInfo> retval = new ArrayList<InputMethodInfo>();
    boolean auxilialyImeAdded = false;
    for (int i = 0; i < imis.size(); ++i) {
        final InputMethodInfo imi = imis.get(i);
        Log.i(TAG, "getDefaultEnabledImes imi " + imi.toString());
        // 【开始】添加以下代码，设置默认IME
        if (imi.getId().equals("com.android.inputmethod.latin/.LatinIME")) {
            retval.clear();
            retval.add(imi);
            Log.i(TAG, "no matter what locale, just set LatinIME as default");
            return retval;
        }
        //【结束】
        if (isDefaultEnabledIme(isSystemReady, imi, context)) {
            retval.add(imi);
            if (imi.isAuxiliaryIme()) {
                auxilialyImeAdded = true;
            }
        }
    }
    if (auxilialyImeAdded) {
        return retval;
    }
    for (int i = 0; i < imis.size(); ++i) {
        final InputMethodInfo imi = imis.get(i);
        if (isSystemAuxilialyImeThatHashAutomaticSubtype(imi)) {
            retval.add(imi);
        }
    }
    return retval;
}

```

frameworks/base/services/java/com/android/server/InputMethodManagerService.java

```java
diff --git a/frameworks/base/services/java/com/android/server/InputMethodManagerService.java b/frameworks/base/services/java/com/android/server/InputMethodManagerService.java
index 16b2d4d..0ff5006 100755
--- a/frameworks/base/services/java/com/android/server/InputMethodManagerService.java
+++ b/frameworks/base/services/java/com/android/server/InputMethodManagerService.java
@@ -724,8 +724,9 @@ public class InputMethodManagerService extends IInputMethodManager.Stub
         InputMethodInfo defIm = null;
         for (InputMethodInfo imi : mMethodList) {
             if (defIm == null) {
-                if (InputMethodUtils.isValidSystemDefaultIme(
-                        mSystemReady, imi, context)) {
+//                if (InputMethodUtils.isValidSystemDefaultIme(
+//                        mSystemReady, imi, context)) {
+                if (imi.getId().equals("com.android.inputmethod.latin/.LatinIME")) {
                     defIm = imi;
                     Slog.i(TAG, "Selected default: " + imi.getId());
                 }

```