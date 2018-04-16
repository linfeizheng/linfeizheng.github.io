---
title: Android运行时权限详解
date: 2018-01-11 12:38:09
tags: 其他
---
### Android系统权限
Android 是一个权限分隔的操作系统，其中每个应用都有其独特的系统标识（Linux 用户 ID 和组 ID）。系统各部分也分隔为不同的标识。Linux 据此将不同的应用以及应用与系统分隔开来。在默认情况下任何应用都没有权限执行对其他应用、操作系统或用户有不利影响的任何操作。包括读取或写入用户的私有数据（例如联系人或电子邮件）、读取或写入其他应用程序的文件、执行网络访问、使设备保持唤醒状态等。

在旧的权限管理系统中，权限仅仅在App安装时询问用户一次，用户同意了这些权限App才能被安装，App一旦安装后后授权不可取消。

Android6.0引入了新的权限模式，将系统权限区分为正常权限和危险权限。开发者在使用到危险权限相关的功能时，不仅需要在Manifest文件中配置，还需要在代码中动态获取权限，如果没有确认获取到权限而直接执行相应所需权限的代码，将导致App崩溃。另外，Android6.0 以上系统,App 退到后台，修改应用权限，再次 App 回到前台，会出现应用新开进程重启。

### 权限的使用
Android App 默认无任何权限，如果需要使用系统权限必须在AndroidManifest.xml文件中声明权限。
``` java
//声明网络使用权限
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example.permission">

<uses-permission android:name="android.permission.INTERNE"/>
    
<application ...>
        ...
</application>

</manifest>
```
如果所需权限为正常权限（即不会对用户隐私或设备操作造成很大风险的权限），系统会自动授予这些权限。

如果所需权限为危险权限（即可能影响用户隐私或设备正常操作的权限），系统会要求用户明确授予这些权限。Android 发出请求的方式取决于系统版本(即targetSdkVersion)：

- 如果运行在 Android 6.0 及以上版本，App targetSdkVersion 大于23，则需要在运行时向用户请求权限，并且需要在 App 使用相关的权限之前检查自身是否已被授予该权限。
- 如果运行在 Android 6.0 以下版本，或App targetSdkVersion 小于23（此时设备可以是Android 6.0 (API level 23)或者更高），则系统会在用户安装则系统会在用户安装App时要求用户授予权限，系统就告诉用户App需要什么权限组。如果App将新权限添加到更新的应用版本，系统会在用户更新应用时要求授予该权限。用户一旦安装应用，他们撤销权限的唯一方式是卸载应用。
#### 正常权限
正常权限涵盖应用需要访问其沙盒外部数据或资源，但对用户隐私或其他应用操作风险很小的区域。例如，设置时区的权限就是正常权限。此类权限都是正常保护的权限，只需要在Manifest文件中简单声明，安装即授权。
- ACCESS_LOCATION_EXTRA_COMMANDS
- ACCESS_NETWORK_STATE
- ACCESS_NOTIFICATION_POLICY
- ACCESS_WIFI_STATE
- BLUETOOTH
- BLUETOOTH_ADMIN
- BROADCAST_STICKY
- CHANGE_NETWORK_STATE
- CHANGE_WIFI_MULTICAST_STATE
- CHANGE_WIFI_STATE
- DISABLE_KEYGUARD
- EXPAND_STATUS_BAR
- GET_PACKAGE_SIZE
- INSTALL_SHORTCUT
- INTERNET
- KILL_BACKGROUND_PROCESSES
- MODIFY_AUDIO_SETTINGS
- NFC
- READ_SYNC_SETTINGS
- READ_SYNC_STATS
- RECEIVE_BOOT_COMPLETED
- REORDER_TASKS
- REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
- REQUEST_INSTALL_PACKAGES
- SET_ALARM
- SET_TIME_ZONE
- SET_WALLPAPER
- SET_WALLPAPER_HINTS
- TRANSMIT_IR
- UNINSTALL_SHORTCUT
- USE_FINGERPRINT
- VIBRATE
- WAKE_LOCK
- WRITE_SYNC_SETTINGS
#### 危险权限

危险权限涵盖应用需要涉及用户隐私信息的数据或资源，或者可能对用户存储的数据或其他应用的操作产生影响的区域。例如，能够读取用户的联系人属于危险权限。

**CALENDAR**
- READ_CALENDAR
- WRITE_CALENDAR

**CAMERA**
- CAMERA

**CONTACTS**
- READ_CONTACTS
- WRITE_CONTACTS
- GET_ACCOUNTS

**LOCATION**
- ACCESS_FINE_LOCATION
- ACCESS_COARSE_LOCATION

**MICROPHONE**
- RECORD_AUDIO

**PHONE**
- READ_PHONE_STATE
- READ_PHONE_NUMBERS
- CALL_PHONE
- ANSWER_PHONE_CALLS (must request at runtime)
- READ_CALL_LOG
- WRITE_CALL_LOG
- ADD_VOICEMAIL
- USE_SIP
- PROCESS_OUTGOING_CALLS

**SENSORS**
- BODY_SENSORS

**SMS**
- SEND_SMS
- RECEIVE_SMS
- READ_SMS
- RECEIVE_WAP_PUSH
- RECEIVE_MMS

**STORAGE**
- READ_EXTERNAL_STORAGE
- WRITE_EXTERNAL_STORAGE

#### 特殊权限
有许多权限其行为方式与正常权限及危险权限都不同。SYSTEM_ALERT_WINDOW 和 WRITE_SETTINGS 特别敏感，因此大多数应用不应该使用它们。如果某应用需要其中一种权限，必须在清单中声明该权限，并且发送请求用户授权的 intent。系统将向用户显示详细信息，以响应该 intent。

### 权限组
所有危险权限都拥有对应权限组，如果运行在 Android 6.0 及以上版本，App targetSdkVersion 大于23，则当用户请求危险权限时系统会发生以下行为：

- 如果应用未拥有Manifest列出的危险权限所在的权限组任一权限，则系统会向用户显示一个对话框，描述应用要访问的权限组。对话框不描述该组内的具体权限。例如，如果应用请求 READ_CONTACTS 权限，系统对话框只说明该应用需要访问设备的联系信息。如果用户批准，系统将向应用授予其请求的权限。
- 如果应用已拥有Manifest列出的危险权限所在的权限组其他任一权限，则系统会立即授予该权限，而无需与用户进行任何交互。例如，如果某应用已经请求并且被授予了 READ_CONTACTS 权限，然后它又请求 WRITE_CONTACTS，系统将立即授予该权限。

### Android O的运行时权限策略变化

- 在 Android O 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用。对于针对Android O的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准。
- 例如，假设某个应用在其清单中列出READ_EXTERNAL_STORAGE和WRITE_EXTERNAL_STORAGE。应用请求READ_EXTERNAL_STORAGE，并且用户授予了该权限，如果该应用针对的是API级别24或更低级别，系统还会同时授予WRITE_EXTERNAL_STORAGE，因为该权限也属于STORAGE权限组并且也在清单中注册过。如果该应用针对的是Android O，则系统此时仅会授予READ_EXTERNAL_STORAGE，不过在该应用以后申请WRITE_EXTERNAL_STORAGE权限时，系统会立即授予该权限，而不会提示用户。
- 我们申请了WRITE_EXTERNAL_STORAGE权限，在Android O之前，我们同时会得到READ_EXTERNAL_STORAGE权限，我们在其它地方涉及到读取存储卡的操作时只需要判断有WRITE_EXTERNAL_STORAGE权限就去读取了。此时应用如果安装在Android O的系统中我们会发现，判断了有WRITE_EXTERNAL_STORAGE权限后去读取存储卡内容时应用崩溃了，原因就是我们没有申请READ_EXTERNAL_STORAGE权限。

### 运行时请求权限

#### 检查权限及兼容

**(1)对于运行在 Android 6.0及以上 App targetSdkVersion 大于23的应用**

- 如果App需要用到危险权限，需要这一权限的操作时都必须检查自己是否拥有该权限。
``` java
int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,Manifest.permission.WRITE_CALENDAR);
```

- 如果应用已经具有了该权限,此方法将返回 PackageManager.PERMISSION_GRANTED，并且应用可以继续操作。如果应用不具有此权限，方法将返回 PERMISSION_DENIED，此时应用应当进行权限申请。

**(2)对于运行在 Android 6.0及以上 App targetSdkVersion 大于23的应用**

- 在App安装时会询问AndroidManifest.xml文件中的权限,用户也可以在设置列表中手动关闭/开启相关权限。

**(3)对于运行在 Android 6.0以下 App targetSdkVersion 大于23的应用**

对于运行在 Android 6.0以下 App targetSdkVersion 大于23的应用，默认情况下是会采取旧的权限机制，然而，一些国产手机在6.0之前就引入了权限管理系统，所以必须对其进行兼容。

- 下面我们来看ActivityCompat.requestPermissions()方法。
``` java
public static void requestPermissions(final @NonNull Activity activity,
        final @NonNull String[] permissions, final @IntRange(from = 0) int requestCode) {
    if (Build.VERSION.SDK_INT >= 23) {
        ActivityCompatApi23.requestPermissions(activity, permissions, requestCode);
    } else if (activity instanceof OnRequestPermissionsResultCallback) {
        Handler handler = new Handler(Looper.getMainLooper());
        handler.post(new Runnable() {
            @Override
            public void run() {
                final int[] grantResults = new int[permissions.length];

                PackageManager packageManager = activity.getPackageManager();
                String packageName = activity.getPackageName();

                final int permissionCount = permissions.length;
                for (int i = 0; i < permissionCount; i++) {
                    grantResults[i] = packageManager.checkPermission(
                            permissions[i], packageName);
                }
                ((OnRequestPermissionsResultCallback) activity).onRequestPermissionsResult(
                        requestCode, permissions, grantResults);
            }
        });
    }
}
```

- 系统版本小于23时，使用packageManager.checkPermission()对权限进行校验，而当App在AndroidManifest.xml中声明权限时，会返回PERMISSION_GRANTED，显然，这一校验方法在Android 6.0以下将失效。

- PermissionChecker.checkSelfPermission() 
PermissionChecker是 Support V4 包下一个专门检查权限的工具类
``` java
public static int checkPermission(@NonNull Context context, @NonNull String permission,
            int pid, int uid, String packageName) {
    if (context.checkPermission(permission, pid, uid) == PackageManager.PERMISSION_DENIED) {
        return PERMISSION_DENIED;
    }

    String op = AppOpsManagerCompat.permissionToOp(permission);
    if (op == null) {
        return PERMISSION_GRANTED;
    }

    if (packageName == null) {
        String[] packageNames = context.getPackageManager().getPackagesForUid(uid);
        if (packageNames == null || packageNames.length <= 0) {
            return PERMISSION_DENIED;
        }
        packageName = packageNames[0];
    }
    if (AppOpsManagerCompat.noteProxyOp(context, op, packageName)
                != AppOpsManagerCompat.MODE_ALLOWED) {
        return PERMISSION_DENIED_APP_OP;
    }
    return PERMISSION_GRANTED;
}
```

checkSelfPermission通过上述四个判断语句进行权限校验

- context.checkPermission()实际上调用的还是上述packageManager.checkPermission()方法进行校验。
- AppOpsManagerCompat.permissionToOp()调用了IMPL.permissionToOp(permission)方法
``` java
private static final AppOpsManagerImpl IMPL;
static {
    if (Build.VERSION.SDK_INT >= 23) {
        IMPL = new AppOpsManager23();
    } else {
        IMPL = new AppOpsManagerImpl();
    }
}
```

- 可以看到，在Android6.0以下设备中，会使用AppOpsManagerImpl,其permissionToOp()方法进行权限检查，其默认返回null,所以PermissionChecker.checkSelfPermission() 同样会失效。
``` java
private static class AppOpsManagerImpl {
    AppOpsManagerImpl() {
    }

    public String permissionToOp(String permission) {
        return null;
    }

    public int noteOp(Context context, String op, int uid, String packageName) {
        return MODE_IGNORED;
    }

    public int noteProxyOp(Context context, String op, String proxiedPackageName) {
        return MODE_IGNORED;
    }
}  
```

- 此外，Google官方还提供了AppOpsManager类来检查权限
``` java
public int checkOp(String op, int uid, String packageName) {
    return checkOp(strOpToOp(op), uid, packageName);
}

@hide 
public int checkOp(int op, int uid, String packageName) {
    try {
        int mode = mService.checkOperation(op, uid, packageName);
        if (mode == MODE_ERRORED) {
            throw new SecurityException(buildSecurityExceptionMsg(op, uid, packageName));
        }
        return mode;
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}
```
- 为了更好的兼容不同厂家的国产手机,建议针对不同的系统版本使用不同的权限校验策略，对于 Android 6.0以上设备，使用PermissionChecker.checkSelfPermission()进行权限校验，对于Android 6.0 以下设备，可通过触发try catch后的危险权限代码检查是否有权限，以期准确校验权限，对用户进行引导，具体可参考AndPermission等第三方库。
### 请求权限
对于App targetSdkVersion 大于23的应用，在应用需要使用的危险权限，必须要进行动态权限申请。Android 提供了多种权限请求方式。调用这些方法将显示一个标准的 Android 对话框。

调用requestPermissions() 方法，可进行动态权限申请，该方法是异步的。在用户响应对话框之后，系统会回调onRequestPermissionsResult。
``` java
if (ContextCompat.checkSelfPermission(thisActivity, Manifest.permission.READ_CONTACTS)
    != PackageManager.PERMISSION_GRANTED) {
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
        Manifest.permission.READ_CONTACTS)) {
    } else {
        ActivityCompat.requestPermissions(thisActivity,
            new String[]{Manifest.permission.READ_CONTACTS},
            MY_PERMISSIONS_REQUEST_READ_CONTACTS);
    }
}  
```

### 处理权限请求回调

在用户响应对话框之后，系统会回调 onRequestPermissionsResult() 方法。

``` java
@Override
public void onRequestPermissionsResult(int requestCode,
    String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            if (grantResults.length > 0
            && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

            } else {

            }
            return;
        }
    }
}
```

### 权限申请被拒绝

如果用户拒绝了某项权限请求，应用应采取适当的操作进行引导，应用应当显示一个对话框，解释应用为什么需要此权限，以及使用该权限有何影响。

当系统要求用户授予权限时，用户可以选择指统不再要求提供该权限。这种情况下，无论应用在什么时候使用 requestPermissions() 请求该权限，系统都会立即拒绝此请求。系统会调用您的 onRequestPermissionsResult() 回调方法，并传递 PERMISSION_DENIED，如果用户再次明确拒绝了权限的请求，系统将采用相同方式操作。这意味着当调用 requestPermissions()时，不一定会出现系统权限请求弹窗。

此时可借助shouldShowRequestPermissionRationale()这个回调方法，如果应用之前请求过此权限但用户拒绝了请求，此方法将返回 true。如果用户拒绝了权限请求，并在权限请求系统对话框中选择了不再询问选项，此方法将返回 false。如果设备默认禁止应用具有该权限，此方法也会返回 false。

**shouldShowRequestPermissionRationale()**

- 望文生义，是否应该显示请求权限的说明。
- 第一次请求权限时，用户拒绝了，调用shouldShowRequestPermissionRationale()后返回true，应该显示一些为什么需要这个权限的说明。
- 用户在第一次拒绝某个权限后，下次再次申请时，授权的dialog中将会出现“不再提醒”选项，一旦选中勾选了，那么下次申请将不会提示用户。
- 第二次请求权限时，用户拒绝了，并选择了“不再提醒”的选项，调用shouldShowRequestPermissionRationale()后返回false。
- 设备的策略禁止当前应用获取这个权限的授权：shouldShowRequestPermissionRationale()返回false 。
- 加这个提醒的好处在于，用户拒绝过一次权限后我们再次申请时可以提醒该权限的重要性，免得再次申请时用户勾选“不再提醒”并决绝，导致下次申请权限直接失败。

### 关于运行时权限的一些建议

**(1)只请求需要的权限，减少请求的次数，或用隐式Intent来让其他的应用来处理。**
- 使用Intent，你不需要设计界面，由第三方的应用来完成所有操作。比如打电话、选择图片等。
- 如果请求权限，你可以完全控制用户体验，自己定义UI。但是用户也可以拒绝权限，就意味着你的应用不能执行这个特殊操作。

**(2)防止一次请求太多的权限或请求次数太多，用户可能对你的应用感到厌烦，在应用启动的时候，最好先请求应用必须的一些权限，非必须权限在使用的时候才请求，建议整理并按照上述分类管理自己的权限：**

- 普通权限（Normal PNermissions）：只需要在Androidmanifest.xml中声明相应的权限，安装即许可。
- 需要运行时申请的权限（Dangerous Permissions）：
- 
必要权限：最好在应用启动的时候，进行请求许可的一些权限（主要是应用中主要功能需要的权限）。

附带权限：不是应用主要功能需要的权限(如：选择图片时，需要读取SD卡权限)。

解释你的应用为什么需要这些权限：在你调用requestPermissions()之前，你为什么需要这个权限。
例如，一个摄影的App可能需要使用定位服务，因为它需要用位置标记照片。一般的用户可能会不理解，他们会困惑为什么他们的App想要知道他的位置。所以在这种情况下，所以你需要在requestpermissions()之前告诉用户你为什么需要这个权限。

**(3)使用兼容库support-v4中的方法**

- PermissionChecker.checkSelfPermission() 或者 ContextCompat.checkSelfPermission()
- ActivityCompat.requestPermissions()
- ActivityCompat.shouldShowRequestPermissionRationale()

### 国产机权限问题整理

- 部分中国厂商生产手机（例如小米）的Rationale功能，在第一次拒绝后，第二次申请时不会返回true，并且会回调申请失败，也就是说在第一次拒绝后默认勾选了不再提示。
- 部分中国厂商生产手机（例如小米、华为）在申请权限时，用户点击确定授权后，还是回调我们申请失败，这个时候其实我们是拥有权限的，所以我们可以在失败的方法中使用AppOpsManager进行权限判断。
- 部分中国厂商生产手机（例如vivo、Oppo）在用户允许权限，并且回调了权限授权成功的方法，但是实际执行代码时并没有这个权限，建议开发者在回调成功的方法中也利用AppOpsManager判断下。
- 在某些手机的Setting中授权后实际检查时还是没有权限，部分执行代码也是没有权限。

### 从系统版本看国产机型的权限申请特点

- 5.0：此时 google 还未着手处理动态权限申请这么个东西，但是我们的小米、魅族等厂商就开始提前设置了强大的权限管理，所以 6.0 权限申请代码在 5.0 上压根不管用，但是说来也简单，5.0 的权限申请对话框激活就是靠触发危险权限代码，然后根据返回值来判断权限是否获取到了（不同手机的返回值判断方式不同，此处需要一一定制）。
- 6.0：国产大部分机型手机的申请权限实际上应该细致地分为申请权限和应用权限 。它们的 ContextCompat.checkSelfPermission(Context, String) 判断是根据是否 AndroidManifest.xml 中声明了该权限来决定返回值，在 AndroidManifest.xml 中声明了权限就返回 true，当然也会有一些会返回 false，这个是申请权限的过程。而真正对话框的弹出是在开发者应用权限的过程中，什么叫做应用权限？就是调用了会触发权限的代码，这个时候就会激活对话框，但是如果仅到这里那就 too young too simple 了，当用户点击拒绝授权时，还是可能会回调授权成功的方法。另外，国产机大部分权限是有三个状态——询问、允许、拒绝——大部分权限都是询问状态，但是有些权限默认是允许状态，有些是拒绝状态，这就导致了调用 ContextCompat.checkSelfPermission(Context, String) 方法时会更畸形，例如小米手机的获取 READ_PHONE_STATE 状态，默认是授予状态。