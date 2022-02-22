# [solved]onActivityResult在LaunchMode为SingleTask/SingleInstance时运行异常

> onActivityResult在LaunchMode为SingleTask/SingleInstance时，会直接返回CANCEL，并且在activity返回时不再触发onActivityResult

## 原因

> 在Android 5.x之前，如果LaunchMode为SingleTask，直接调用onActivityResult，并返回Cancel结果
> 在Android 5.x之后，会抹除SingleTask/SingleInstance Flag，然后正常调用

## 源码

** \<Android5.x **

```java
if (sourceRecord == null) {
    // This activity is not being started from another...  in this
    // case we -always- start a new task.
    if ((launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
        Slog.w(TAG, "startActivity called from non-Activity context; forcing " +
                "Intent.FLAG_ACTIVITY_NEW_TASK for: " + intent);
        launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
    }
} else if (sourceRecord.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
    // The original activity who is starting us is running as a single
    // instance...  this new activity it is starting must go on its
    // own task.
    launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
} else if (r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE
        || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK) {
    // The activity being started is a single instance...  it always
    // gets launched into its own task.
    launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
}
// ......
if (r.resultTo != null && (launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
    // For whatever reason this activity is being launched into a new
    // task...  yet the caller has requested a result back.  Well, that
    // is pretty messed up, so instead immediately send back a cancel
    // and let the new task continue launched as normal without a
    // dependency on its originator.
    Slog.w(TAG, "Activity is launching as a new task, so cancelling activity result.");
    r.resultTo.task.stack.sendActivityResultLocked(-1,
            r.resultTo, r.resultWho, r.requestCode,
        Activity.RESULT_CANCELED, null);
    r.resultTo = null;
}
```

** \>Android5.x **

```java
final boolean launchSingleTop = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TOP;
final boolean launchSingleInstance = r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE;
final boolean launchSingleTask = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK;
int launchFlags = intent.getFlags();
if ((launchFlags & Intent.FLAG_ACTIVITY_NEW_DOCUMENT) != 0 &&
        (launchSingleInstance || launchSingleTask)) {
    // We have a conflict between the Intent and the Activity manifest, manifest wins.
    Slog.i(TAG, "Ignoring FLAG_ACTIVITY_NEW_DOCUMENT, launchMode is " +
            "\"singleInstance\" or \"singleTask\"");
    launchFlags &=
            ~(Intent.FLAG_ACTIVITY_NEW_DOCUMENT | Intent.FLAG_ACTIVITY_MULTIPLE_TASK);
} else {
    switch (r.info.documentLaunchMode) {
        case ActivityInfo.DOCUMENT_LAUNCH_NONE:
            break;
        case ActivityInfo.DOCUMENT_LAUNCH_INTO_EXISTING:
            launchFlags |= Intent.FLAG_ACTIVITY_NEW_DOCUMENT;
            break;
        case ActivityInfo.DOCUMENT_LAUNCH_ALWAYS:
            launchFlags |= Intent.FLAG_ACTIVITY_NEW_DOCUMENT;
            break;
        case ActivityInfo.DOCUMENT_LAUNCH_NEVER:
            launchFlags &= ~Intent.FLAG_ACTIVITY_MULTIPLE_TASK;
            break;
    }
}
final boolean launchTaskBehind = r.mLaunchTaskBehind
        && !launchSingleTask && !launchSingleInstance
        && (launchFlags & Intent.FLAG_ACTIVITY_NEW_DOCUMENT) != 0;
if (r.resultTo != null && (launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
    // For whatever reason this activity is being launched into a new
    // task...  yet the caller has requested a result back.  Well, that
    // is pretty messed up, so instead immediately send back a cancel
    // and let the new task continue launched as normal without a
    // dependency on its originator.
    Slog.w(TAG, "Activity is launching as a new task, so cancelling activity result.");
    r.resultTo.task.stack.sendActivityResultLocked(-1,
            r.resultTo, r.resultWho, r.requestCode,
            Activity.RESULT_CANCELED, null);
    r.resultTo = null;
}
```

## 参考
1. [onActivityResult With launchMode="singleTask"?](https://stackoverflow.com/questions/8960072/onactivityresult-with-launchmode-singletask)





