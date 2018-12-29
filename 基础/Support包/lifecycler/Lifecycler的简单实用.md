```java
// Activity中调用
getLifecycle().addObserver(new GenericLifecycleObserver() {
    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        Log.d("FIEND", "onStateChanged1" + event.name());
    }
});

// 如果可以拿到Lifecycle，可以在非Activity中调用addObserver方法
```



* 如果在非onCreate方法中调用addObserver，Lifecycler会从onCreate开始回调到最终的状态
  * eg：在onStart()方法中调用addObserver，onStateChanged也会从onCreate状态开始分发

