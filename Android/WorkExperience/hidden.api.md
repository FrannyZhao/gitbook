[toc]

# 获得安卓隐藏API

## 静默获得录屏权限

```java
    @RequiresApi(21)
    private void setMediaProjectionIntentWithoutPermission(Context context) {
        if (mMediaProjectionIntent == null) {
            String mPackageName = context.getPackageName();;
            int mUid = -1;
            PackageManager packageManager = context.getPackageManager();
            ApplicationInfo aInfo;
            try {
                aInfo = packageManager.getApplicationInfo(mPackageName, 0);
                mUid = aInfo.uid;
            } catch (PackageManager.NameNotFoundException e) {
                Log.e(TAG, "unable to look up package name", e);
            }
            Log.i(TAG, "requestPermissions mPackageName " + mPackageName + ", mUid " + mUid);
            try {
                Class<?> serviceManager = Class.forName("android.os.ServiceManager");
                IBinder serviceBinder = (IBinder)serviceManager.getMethod("getService", String.class)
                        .invoke(serviceManager, "media_projection");
                Class<?> stub = Class.forName("android.media.projection.IMediaProjectionManager$Stub");
                Object projectionManagerService = stub.getMethod("asInterface", IBinder.class).invoke(stub, serviceBinder);
                Method hasProjectionPermission = projectionManagerService.getClass()
                        .getMethod("hasProjectionPermission", int.class, String.class);
                boolean result = (boolean)hasProjectionPermission.invoke(projectionManagerService, mUid, mPackageName);
                Log.i(TAG, "hasProjectionPermission " + result);

                Method createProjection = projectionManagerService.getClass()
                        .getMethod("createProjection", int.class, String.class, int.class, boolean.class);
                Class<?> projectionClass = Class.forName("android.media.projection.IMediaProjection");
                Method asBinderMethod = projectionClass.getMethod("asBinder");

                Object projectionObj = createProjection.invoke(projectionManagerService, mUid, mPackageName, 0, true);
                IBinder projectionBinder = (IBinder)asBinderMethod.invoke(projectionObj);
                Intent intent = new Intent();
                Method putBinderExtra = intent.getClass().getMethod("putExtra", String.class, IBinder.class);
                putBinderExtra.invoke(intent, "android.media.projection.extra.EXTRA_MEDIA_PROJECTION", projectionBinder);
                Log.i(TAG, "successfully get media projection intent");
                mMediaProjectionIntent = intent;
            } catch (ClassNotFoundException | ClassCastException | NoSuchMethodException | SecurityException
                    | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
                Log.w(TAG, e.toString());
                e.printStackTrace();
            }
        }
    }
```



## 判断是否首次启动或者factory reset后首次启动

```java
boolean isFirstBoot = false;
        IBinder pmsIBinder = ServiceManager.getService("package");
        try {
            Class<?> stub = Class.forName("android.content.pm.IPackageManager$Stub");
            Object pms = stub.getMethod("asInterface", IBinder.class).invoke(stub, pmsIBinder);
            isFirstBoot = (boolean)pms.getClass().getMethod("isFirstBoot").invoke(pms);
            Log.w(TAG, "setVolumeMaxWhenFirstBoot isFirstBoot " + isFirstBoot);
        } catch (ClassNotFoundException | IllegalAccessException | InvocationTargetException | NoSuchMethodException e) {
            e.printStackTrace();
            Log.w(TAG, "setVolumeMaxWhenFirstBoot error " + e.toString());
        }
```

