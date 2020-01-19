[toc]

# content provider

需求： 跨进程分享一个shared preference里面存的值

## 分享者

### AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.franny.test">
    
    <application
        ...>
...
        <provider android:name=".FrannyTestProvider" android:authorities="com.franny.test"
            android:exported="true"/>

    </application>
    <permission
        android:name="com.franny.test.READ_PERMISSION"
        android:label="Franny provider read permission"
        android:protectionLevel="normal"
        />
</manifest>
```

### ContentProvider

```java
package com.franny.test;

import android.content.ContentProvider;
import android.content.ContentValues;
import android.content.UriMatcher;
import android.database.Cursor;
import android.net.Uri;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;

public class FrannyTestProvider extends ContentProvider {
    private static final String TAG = FrannyTestProvider.class.getSimpleName();
    private UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    private static final String AUTHORITY = "com.franny.test";
    private static final int DEVICE_ID = 1;
    @Override
    public boolean onCreate() {
        uriMatcher.addURI(AUTHORITY, "device_id", DEVICE_ID);
        return true;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        return null;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        String result = "default";
        switch (uriMatcher.match(uri)) {
            case DEVICE_ID:
                result = "device";
                // todo 此处改为要分享给另一个应用的值
                break;
            default:
                break;
        }
        return result;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        return null;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }
}

```



## 获得者

### AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.m.launcher">
    <uses-permission android:name="com.franny.test.READ_PERMISSION"/>

    <application
                 ...
```

### XXActivity.java

```JAVA
Timber.i("shared value: " + getContentResolver().getType(Uri.parse("content://com.m.iot/device_id")));
```

