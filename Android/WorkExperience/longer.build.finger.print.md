[TOC]

# Android Longer Build Finger Print

编译报错：

> error: ro.build.fingerprint cannot exceed 91 bytes

需要修改系统定义的PROP_VALUE_MAX和PROPERTY_VALUE_MAX。

每一版安卓代码不太一样，这里给的是安卓**8.1.0**的代码修改。其他版本只需要在http://androidxref.com/对应的版本里面搜索这两个关键字```PROP_VALUE_MAX, PROPERTY_VALUE_MAX```的定义，修改对应的数值。

### build/make

```python
diff --git a/tools/post_process_props.py b/tools/post_process_props.py
index 9355e4b..91bcc76 100755
--- a/tools/post_process_props.py
+++ b/tools/post_process_props.py
@@ -22,7 +22,7 @@ import sys
 # See PROP_VALUE_MAX in system_properties.h.
 # The constant in system_properties.h includes the terminating NUL,
 # so we decrease the value by 1 here.
-PROP_VALUE_MAX = 91
+PROP_VALUE_MAX = 99
 
 # Put the modifications that you need to make into the /system/build.prop into this
 # function. The prop object has get(name) and put(name,value) methods.
```

### bionic

```c
diff --git a/libc/include/sys/system_properties.h b/libc/include/sys/system_properties.h
index d075859..e83d426 100644
--- a/libc/include/sys/system_properties.h
+++ b/libc/include/sys/system_properties.h
@@ -38,7 +38,7 @@ __BEGIN_DECLS
 
 typedef struct prop_info prop_info;
 
-#define PROP_VALUE_MAX  92
+#define PROP_VALUE_MAX  100
 
 /*
  * Sets system property `key` to `value`, creating the system property if it doesn't already exist.
```

### system/bt

```c
diff --git a/osi/include/properties.h b/osi/include/properties.h
index ececef5..508aa00 100644
--- a/osi/include/properties.h
+++ b/osi/include/properties.h
@@ -20,7 +20,7 @@
 
 #include <cstdint>
 #if defined(OS_GENERIC)
-#define PROPERTY_VALUE_MAX 92
+#define PROPERTY_VALUE_MAX 100
 #else
 #include <cutils/properties.h>
 #endif  // defined(OS_GENERIC)
```

-------

改了这些还会有编译报错：

> frameworks/native/cmds/installd/installd.cpp:43:1: error: static_assert failed "Size mismatch."
> static_assert(kPropertyValueMax == PROPERTY_VALUE_MAX, "Size mismatch.");

再修改kPropertyValueMax的定义：

### frameworks/native

```c
diff --git a/cmds/installd/installd_deps.h b/cmds/installd/installd_deps.h
index 5093178..ca467fe 100644
--- a/cmds/installd/installd_deps.h
+++ b/cmds/installd/installd_deps.h
@@ -34,7 +34,7 @@ extern int get_property(const char *key,
                         const char *default_value);
 // Size constants. Should be checked to be equal to the cutils requirements.
 constexpr size_t kPropertyKeyMax = 32u;
-constexpr size_t kPropertyValueMax = 92u;
+constexpr size_t kPropertyValueMax = 100u;
 
 // Compute the output path for dex2oat.
 extern bool calculate_oat_file_path(char path[PKG_PATH_MAX],
```

