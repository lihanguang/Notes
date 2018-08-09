#### 目录
* 介绍
* 关键对象
* 简单流程分析
* 线程切换
* 运算符

#### Rxjava1.x介绍
RxJava是Reactive Extensions的Java VM实现：一个通过使用可观察序列来编写异步和基于事件的程序的库。

它扩展了观察者模式以支持数据/事件序列，并添加了运算符，使您可以声明性地组合序列，同时抽象出对低级线程，同步，线程安全性和并发数据结构等问题的关注。

以上来自Gihub中Rxjava的介绍的Google翻译。

#### 关键对象
* Observable：被观察者，位于事件上流，负责发送事件
``` 
public class Observable<T> {

    //唯一构造方法
    protected Observable(OnSubscribe<T> f) {
            this.onSubscribe = f;
    }
    
    //绑定方法，与Subscriber绑定
    public final Subscription subscribe(Subscriber<? super T> subscriber) {
        return Observable.subscribe(subscriber, this);
    }
}

```
* Subscription：订阅对象，查看订阅情况（是否还在订阅者），取消订阅
```
public interface Subscription {

    /**
     * 停止订阅
     */
    void unsubscribe();

    /**
     * 是否订阅
     */
    boolean isUnsubscribed();

}
```
* Observer：观察者，位于事件下游，接受事件并处理
```
public interface Observer<T> {

    //已完成
    void onCompleted();

    //错误
    void onError(Throwable e);

    //接受事件
    void onNext(T t);

}
```
* Subscriber：关联观察者和订阅对象，负责发送事件和管理订阅的生命周期，这是一个抽象类，Observer的相关方法（onNext、onError、onCompleted）没有实现，具体实现需要参考具体实现类（比如SafeSubscriber，下面有介绍）
```
public abstract class Subscriber<T> implements Observer<T>, Subscription {

    //取消订阅
    @Override
    public final void unsubscribe() {
        subscriptions.unsubscribe();
    }

    //是否在订阅中
    @Override
    public final boolean isUnsubscribed() {
        return subscriptions.isUnsubscribed();
    }

    //订阅开始
    public void onStart() {
        // do nothing by default
    }

```
* ActionN：动作（没有返回值），N表示有N个数据
```
public interface Action1<T> extends Action {
    void call(T t);
}
```
* FunN：动作（有返回值），N表示有N个数据
```
public interface Func1<T, R> extends Function {
    R call(T t);
}
```
* Schulder：线程调度器
* Worker：任务执行器，执行具体的任务
#### 简单流程分析
以下面的代码为例：
```
Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("aaa");
                subscriber.onNext("bbb");
                subscriber.onCompleted();
            }
        }).subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                System.out.print(s);
            }
        });
```

首先调用
```
public static <T> Observable<T> create(OnSubscribe<T> f) {
        return new Observable<T>(RxJavaHooks.onCreate(f));
} 

protected Observable(OnSubscribe<T> f) {
        this.onSubscribe = f;
}
```
来创建一个Observable，只是简单的把OnSubsriber对象赋值。然后调用subscribe方法来关联观察者和被观察者
```
public final Subscription subscribe(Subscriber<? super T> subscriber) {
    return Observable.subscribe(subscriber, this);
}

//绑定方法都会通过该方法来执行
static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {
    ... 
    // 运行在subscribe线程,Subscriber中默认实现为空，需要时可以覆盖
    subscriber.onStart();

    // 封装为安全的subsriber，避免onNext、onError和onComplete的调用混乱，
    //比如onError和onComplete之后不能调用onNext，onComplete和onError只能调用一个.
    if (!(subscriber instanceof SafeSubscriber)) {
        subscriber = new SafeSubscriber<T>(subscriber);
    }

    try {
        // 无视Hook，调用observable.onSubscribe（也就是构造方法传入的对象）的call方法，
        //也就是我们实现的call方法，用来发送事件。
        //这样我们的事件就从上游出发了。
        RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);
        return RxJavaHooks.onObservableReturn(subscriber);
    } catch (Throwable e) {
        ...
        //异常
        return Subscriptions.unsubscribed();
    }
}
```
call方法中传入的subscriber是经过封装的SafeSubscriber。
```
public class SafeSubscriber<T> extends Subscriber<T> {
    boolean done; //标志订阅是否已经完成，调用onError和onComplete都会置true

    @Override
    public void onCompleted() {
        if (!done) { //对该标志进行判断，若已经完成则不会执行逻辑
            done = true;
            try {
                actual.onCompleted();
            } catch (Throwable e) {
                ...
                //异常处理
            } finally { // NOPMD
                try {
                    //onCompleted之后会主动解除订阅避免内存泄漏，onError逻辑基本一样，这里就不贴代码了
                    unsubscribe();
                } catch (Throwable e) {
                    RxJavaHooks.onError(e);
                    throw new UnsubscribeFailedException(e.getMessage(), e);
                }
            }
        }
    }

    @Override
    public void onNext(T t) {
        try {
            if (!done) { 
                actual.onNext(t);//将事件发送给下游的观察者
            }
        } catch (Throwable e) {
            Exceptions.throwOrReport(e, this);
        }
    }

```
* 没有线程切换订阅过程比较简单
* 因为在单线程中运行，所以Subscription的订阅生命周期管理无法生效（因为从订阅开始到订阅结束都自己处理了），其流程毁在线程切换中讨论。

#### 线程切换
* Rxjava可以很方便的进行线程切换
* 其通过引入Scheduler来进行线程切换
* Rxjava的线程切换如下：
```
Observable.create(new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> subscriber) {
            subscriber.onNext("aaa");
            subscriber.onNext("bbb");
            subscriber.onCompleted();
        }
    }).subscribeOn(Schedulers.computation())
            .observeOn(Schedulers.immediate())
            .subscribe(new Subscriber<String>() {
                @Override
                public void onCompleted() {

                }

                @Override
                public void onError(Throwable e) {

                }

                @Override
                public void onNext(String s) {
                    System.out.print(s);
                }
            });
```
只是在简单使用的基础上添加了这两行代码。
```
.subscribeOn(Schedulers.computation()) //下面就分析Scheduler为computation()的情况
.observeOn(Schedulers.immediate())
```
* 首先分析subscribeOn过程，进入subscribeOn方法中
```
public final Observable<T> subscribeOn(Scheduler scheduler) {
    return subscribeOn(scheduler, !(this.onSubscribe instanceof OnSubscribeCreate));
}
```
然后
```
public final Observable<T> subscribeOn(Scheduler scheduler, boolean requestOn) {
    if (this instanceof ScalarSynchronousObservable) {
        return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
    }
    //忽视上面的特殊情况
    return unsafeCreate(new OperatorSubscribeOn<T>(this, scheduler, requestOn));
}
```
可以看到scheduler被封装到了OperatorSubscribeOn当中
```
//继承于OnSubscribe，所以subscribe过程会调用其call方法。
public final class OperatorSubscribeOn<T> implements OnSubscribe<T>{
    @Override
    public void call(final Subscriber<? super T> subscriber) {
        final Worker inner = scheduler.createWorker();//根据不同的scheduler会得到不同的Worker

        SubscribeOnSubscriber<T> parent = new SubscribeOnSubscriber<T>(subscriber, requestOn, inner, source);
        subscriber.add(parent);
        subscriber.add(inner);

        inner.schedule(parent); //调用Worker的schedule方法来进入到线程池中运行
    }
}
```
以Schedulers.computation()为例
```
//computation()最终会进入RxJavaSchedulersHook中
public static Scheduler createComputationScheduler(ThreadFactory threadFactory) {
    if (threadFactory == null) {
        throw new NullPointerException("threadFactory == null");
    }
    return new EventLoopsScheduler(threadFactory);
}

public final class EventLoopsScheduler extends Scheduler implements SchedulerLifecycle {
    @Override
    public Worker createWorker() {
        return new EventLoopWorker(pool.get().getEventLoop()); //
    }
}
```
EventLoopWorker是EventLoopSchduler的内部类
```
static final class EventLoopWorker extends Scheduler.Worker {
        private final SubscriptionList serial = new SubscriptionList();
        private final CompositeSubscription timed = new CompositeSubscription();
        private final SubscriptionList both = new SubscriptionList(serial, timed);
        private final PoolWorker poolWorker;

        EventLoopWorker(PoolWorker poolWorker) {
            this.poolWorker = poolWorker;
        }

        @Override
        public void unsubscribe() {
            both.unsubscribe();
        }

        @Override
        public boolean isUnsubscribed() {
            return both.isUnsubscribed();
        }

        @Override
        public Subscription schedule(final Action0 action) {
            if (isUnsubscribed()) {
                return Subscriptions.unsubscribed();
            }

            //可以看到最后交给了poolWorker来执行
            return poolWorker.scheduleActual(new Action0() {
                @Override
                public void call() {
                    if (isUnsubscribed()) {
                        return;
                    }
                    action.call();
                }
            }, 0, null, serial);
        }

        @Override
        public Subscription schedule(final Action0 action, long delayTime, TimeUnit unit) {
            if (isUnsubscribed()) {
                return Subscriptions.unsubscribed();
            }
            
            //可以看到最后交给了poolWorker来执行
            return poolWorker.scheduleActual(new Action0() {
                @Override
                public void call() {
                    if (isUnsubscribed()) {
                        return;
                    }
                    action.call();
                }
            }, delayTime, unit, timed);
        }
    }
```
poolWorker是PoolWorker的对象，而PoolWorker继承于NewThreadWorker
```
public class NewThreadWorker extends Scheduler.Worker implements Subscription {
   
    public NewThreadWorker(ThreadFactory threadFactory) {
        //创建只有一个核心线程的线程池，Rxjava中会对Java的线程池进行一定的封装，
        //而一般只会使用Java的ScheduledThreadPool，因为Rxjava需要提供延时的执行方法。
        ScheduledExecutorService exec = Executors.newScheduledThreadPool(1, threadFactory);
        boolean cancelSupported = tryEnableCancelPolicy(exec);
        if (!cancelSupported && exec instanceof ScheduledThreadPoolExecutor) {
            registerExecutor((ScheduledThreadPoolExecutor)exec);
        }
        executor = exec;
    }

    @Override
    public Subscription schedule(final Action0 action) {
        return schedule(action, 0, null);
    }

    @Override
    public Subscription schedule(final Action0 action, long delayTime, TimeUnit unit) {
        if (isUnsubscribed) {
            return Subscriptions.unsubscribed();
        }
        return scheduleActual(action, delayTime, unit);
    }

    public ScheduledAction scheduleActual(final Action0 action, long delayTime, TimeUnit unit) {
        Action0 decoratedAction = RxJavaHooks.onScheduledAction(action);
        ScheduledAction run = new ScheduledAction(decoratedAction); //SchduledAction实现了Runnable接口和Subscription接口
        Future<?> f;
        if (delayTime <= 0) {
            f = executor.submit(run);
        } else {
            f = executor.schedule(run, delayTime, unit);
        }
        run.add(f);

        return run;
    }
   
    @Override
    public void unsubscribe() {
        isUnsubscribed = true;
        executor.shutdownNow();
        deregisterExecutor(executor);
    }

    @Override
    public boolean isUnsubscribed() {
        return isUnsubscribed;
    }
}
```
SchduledAction实现了Runnable接口和Subscription接口，所以它会被放到线程池中运行，并管理subscribe的生命周期
```
public final class ScheduledAction extends AtomicReference<Thread> implements Runnable, Subscription {
    private static final long serialVersionUID = -3962399486978279857L;
    final SubscriptionList cancel;
    final Action0 action;

    public ScheduledAction(Action0 action) {
        this.action = action;
        this.cancel = new SubscriptionList();
    }
    //适配不同的Subsription
    public ScheduledAction(Action0 action, CompositeSubscription parent) {
        this.action = action;
        this.cancel = new SubscriptionList(new Remover(this, parent));
    }
    //适配不同的Subsription
    public ScheduledAction(Action0 action, SubscriptionList parent) {
        this.action = action;
        this.cancel = new SubscriptionList(new Remover2(this, parent));
    }

    @Override
    public void run() {
        try {
            lazySet(Thread.currentThread());
            action.call(); //执行具体操作
        } catch (OnErrorNotImplementedException e) {
            signalError(new IllegalStateException("Exception thrown on Scheduler.Worker thread. Add `onError` handling.", e));
        } catch (Throwable e) {
            signalError(new IllegalStateException("Fatal Exception thrown on Scheduler.Worker thread.", e));
        } finally {
            unsubscribe();
        }
    }

    @Override
    public boolean isUnsubscribed() {
        return cancel.isUnsubscribed();
    }

    @Override
    public void unsubscribe() {
        if (!cancel.isUnsubscribed()) {
            cancel.unsubscribe();
        }
    }
    
    public void add(final Future<?> f) {
        cancel.add(new FutureCompleter(f)); //将Future适配为Subscription
    }
    
    //对Future进行Subsription封装，托管其的生命周期
    final class FutureCompleter implements Subscription {
        private final Future<?> f;

        FutureCompleter(Future<?> f) {
            this.f = f;
        }

        @Override
        public void unsubscribe() {
            if (ScheduledAction.this.get() != Thread.currentThread()) {
                f.cancel(true);
            } else {
                f.cancel(false);
            }
        }
        @Override
        public boolean isUnsubscribed() {
            return f.isCancelled();
        }
    }
}
```
* 至此，subscribeOn的过程就结束了，这里需要说一下不同的Scheduler的主要区别在于线程池的线程数量不同，io调度器使用的是无限线程池，新任务会开启一个新的线程，compution则使用了固定线程池（线程数量小于cpu数量），这里的线程池是不是指Java中的线程池，而是封装了Java线程池的线程池。
* newThread和io都使用了无限线程，但是newThread没有封装线程池，所以在执行db、网络等io操作时io调度器是最好的选择
* 接下来分析observeOn过程
```
public final Observable<T> observeOn(Scheduler scheduler) {
    return observeOn(scheduler, RxRingBuffer.SIZE); //SIZE是结果的队列容量，因为不同线程中运行需要考虑背压
}
```
辗转之后，最终会调用
```
public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
    if (this instanceof ScalarSynchronousObservable) {
        return ((ScalarSynchronousObservable<T>)this).scalarScheduleOn(scheduler);
    }
    //最终会通过lift操作符来实现
    return lift(new OperatorObserveOn<T>(scheduler, delayError, bufferSize));
}
```
OperatorObserveOn是一个实现了Operator<T, T>接口的类，Operator具体为
```
public interface Operator<R, T> extends Func1<Subscriber<? super R>, Subscriber<? super T>> {
    // cover for generics insanity
}
```
而Func1是一个带有返回值的call方法的接口
```
public interface Func1<T, R> extends Function {
    R call(T t);
}
```
因为上面的实现中使用了<T, T>，所以它是转会为自己的操作。而OperatorObserveOn的具体实现为
```
public final class OperatorObserveOn<T> implements Operator<T, T> {
    @Override
    public Subscriber<? super T> call(Subscriber<? super T> child) {
        if (scheduler instanceof ImmediateScheduler) {
            // avoid overhead, execute directly
            return child;
        } else if (scheduler instanceof TrampolineScheduler) {
            // avoid overhead, execute directly
            return child;
        } else {
            //先忽视上面的特殊情况
            ObserveOnSubscriber<T> parent = new ObserveOnSubscriber<T>(scheduler, child, delayError, bufferSize);
            parent.init(); //下面分析
            return parent;
        }
    }
}
```
ObserveOnSubscriber是OperatorObserveOn的一个内部类
```
static final class ObserveOnSubscriber<T> extends Subscriber<T> implements Action0 {
    //观察继承结构，可以发现它是继承于Subscriber，也就是对我们传入的Subscriber的封装，同时也实现了Action0的接口，所以他会被放入Schulder中运行
    public ObserveOnSubscriber(Scheduler scheduler, Subscriber<? super T> child, boolean delayError, int bufferSize) {
            this.child = child;
            this.recursiveScheduler = scheduler.createWorker();
            this.delayError = delayError;
            int calculatedSize = (bufferSize > 0) ? bufferSize : RxRingBuffer.SIZE;
            // this formula calculates the 75% of the bufferSize, rounded up to the next integer
            this.limit = calculatedSize - (calculatedSize >> 2);
            //这里创建一个队列用来保存上流传入的事件，其大小就是我们传入的buffersize
            if (UnsafeAccess.isUnsafeAvailable()) {
                queue = new SpscArrayQueue<Object>(calculatedSize);
            } else {
                queue = new SpscAtomicArrayQueue<Object>(calculatedSize);
            }
            // signal that this is an async operator capable of receiving this many
            request(calculatedSize);
        }

        @Override
        public void onNext(final T t) { //因为封装了我们传入的Subscriber，所以这里会接收到上流传入的事件
            if (isUnsubscribed() || finished) {
                return;
            }
            if (!queue.offer(NotificationLite.next(t)))  {//这里不是直接处理事件，而是把他保存在一个队列当中
                onError(new MissingBackpressureException());//如果队列容量已经达到最大值则会抛出异常
                return;
            }
            schedule(); //下面分析
        }

        protected void schedule() {
            if (counter.getAndIncrement() == 0) {
                //这里会进入到调度器中进行执行，与之前分析subscribeOn过程一样，
                //他会调用下面的call方法来对上流的事件进行处理
                recursiveScheduler.schedule(this); 
            }
        }

        // only execute this from schedule()
        @Override
        public void call() {
            
            ...
            for (;;) {
                long requestAmount = requested.get();

                while (requestAmount != currentEmission) {
                    boolean done = finished;
                    Object v = q.poll(); //获取一个上流事件
                    
                    ...

                    localChild.onNext(NotificationLite.<T>getValue(v));//执行一次事件，localChild也就是我们传入的Subscriber，所以这里就到了我们接收结果的地方

                    ...
                }

                ...
            }
        }
}
```
接着，lift操作会完成最后的封装
```
public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
    return unsafeCreate(new OnSubscribeLift<T, R>(onSubscribe, operator));
}
```
OnSubscribeLift继承于OnSubscribe
```
public final class OnSubscribeLift<T, R> implements OnSubscribe<R> {

    @Override
    public void call(Subscriber<? super R> o) {
        try {
            Subscriber<? super T> st = RxJavaHooks.onObservableLift(operator).call(o); //完成转换
            try {
                // new Subscriber created and being subscribed with so 'onStart' it
                st.onStart();
                parent.call(st); //执行
            } catch (Throwable e) {
                ...
            }
        } catch (Throwable e) {
            ...
        }
    }
}
```
* 至此，observeOn过程就结束了。对于这个observeOn过程，其根本原理是对Subscriber（也就是我们传入的Subscriber）的封装，然后通过将上流传来的事件保存在队列中，然后在开启线程去逐个执行（或者并发执行），这样就达到了线程切换的目的。

#### 操作符
* Rxjava提供了非常多的操作符，经常使用的就有map、filter、flatMap等，这里就只分析上面这三种。
* map操作符
```
public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
    return unsafeCreate(new OnSubscribeMap<T, R>(this, func));
}
```
可以看到map操作符被封装到了OnSubscribeMap中
```
public final class OnSubscribeMap<T, R> implements OnSubscribe<R> {
    final Observable<T> source;
    final Func1<? super T, ? extends R> transformer;
    public OnSubscribeMap(Observable<T> source, Func1<? super T, ? extends R> transformer) {
        this.source = source;
        this.transformer = transformer;
    }

    @Override
    public void call(final Subscriber<? super R> o) {
        MapSubscriber<T, R> parent = new MapSubscriber<T, R>(o, transformer);
        o.add(parent);
        source.unsafeSubscribe(parent); //使用不安全的订阅，因为MapSubscriber提供了安全的订阅
    }

    static final class MapSubscriber<T, R> extends Subscriber<T> {
        final Subscriber<? super R> actual;
        final Func1<? super T, ? extends R> mapper;
        boolean done;
        public MapSubscriber(Subscriber<? super R> actual, Func1<? super T, ? extends R> mapper) {
            this.actual = actual;
            this.mapper = mapper;
        }

        @Override
        public void onNext(T t) {
            R result;

            try {
                result = mapper.call(t); //通过Fun1进行转换
            } catch (Throwable ex) {
                Exceptions.throwIfFatal(ex);
                unsubscribe();
                onError(OnErrorThrowable.addValueAsLastCause(ex, t));
                return;
            }

            actual.onNext(result);//将转换的结果发送给下游
        }

        @Override
        public void onError(Throwable e) {
            if (done) {
                RxJavaHooks.onError(e);
                return;
            }
            done = true;

            actual.onError(e);
        }


        @Override
        public void onCompleted() {
            if (done) {
                return;
            }
            actual.onCompleted();
        }

        @Override
        public void setProducer(Producer p) {
            actual.setProducer(p);
        }
    }

}
```
* fliter操作符：和map的实现类似，主要区别在于Fun1的泛型为<? super T, Boolean>，表示要转换为Boolean，并根本转换结果来判断是否要发送给下游
```
public void onNext(T t) {
    boolean result;

    try {
        result = predicate.call(t); //获得转换结果
    } catch (Throwable ex) {
        Exceptions.throwIfFatal(ex);
        unsubscribe();
        onError(OnErrorThrowable.addValueAsLastCause(ex, t));
        return;
    }

    if (result) { //判断是否要发送给下游
        actual.onNext(t);
    } else {
        request(1);
    }
}
```
* flatMap：和map类似，但是它是将一个事件转换为一个被观察者。
```
public final <R> Observable<R> flatMap(Func1<? super T, ? extends Observable<? extends R>> func) {
    if (getClass() == ScalarSynchronousObservable.class) {
        return ((ScalarSynchronousObservable<T>)this).scalarFlatMap(func);
    }
    return merge(map(func));
}
```
可以看到首先会先调用map来使? super T转换为? extends Observable<? extends R>，所以转换的结果解释Observable<? extends Observable<? extends R>>，然后使用merge操作符
```
public static <T> Observable<T> merge(Observable<? extends Observable<? extends T>> source) {
    if (source.getClass() == ScalarSynchronousObservable.class) {
        return ((ScalarSynchronousObservable<T>)source).scalarFlatMap((Func1)UtilityFunctions.identity());
    }
    //忽视特殊情况
    return source.lift(OperatorMerge.<T>instance(false));
}
```
merge操作符也使用了lift，而lift则是需要一个Operator，Operator会对我们传入的Subscriber进行转换
```
public final class OperatorMerge<T> implements Operator<T, Observable<? extends T>> {
    //观察继承树可以看到这个操作是将T转换为Observable<? extends T>

    final boolean delayErrors;
    final int maxConcurrent;

    static final class HolderNoDelay {
        static final OperatorMerge<Object> INSTANCE = new OperatorMerge<Object>(false, Integer.MAX_VALUE);
    }
    static final class HolderDelayErrors {
        static final OperatorMerge<Object> INSTANCE = new OperatorMerge<Object>(true, Integer.MAX_VALUE);
    }
    
    //这里采用了延迟加载的策略
    public static <T> OperatorMerge<T> instance(boolean delayErrors) {
        if (delayErrors) {
            return (OperatorMerge<T>)HolderDelayErrors.INSTANCE;
        }
        return (OperatorMerge<T>)HolderNoDelay.INSTANCE;
    }
    
    public static <T> OperatorMerge<T> instance(boolean delayErrors, int maxConcurrent) {
        if (maxConcurrent <= 0) {
            throw new IllegalArgumentException("maxConcurrent > 0 required but it was " + maxConcurrent);
        }
        if (maxConcurrent == Integer.MAX_VALUE) {
            return instance(delayErrors);
        }
        return new OperatorMerge<T>(delayErrors, maxConcurrent);
    }
    
    @Override
    public Subscriber<Observable<? extends T>> call(final Subscriber<? super T> child) {
        //转换过程
        MergeSubscriber<T> subscriber = new MergeSubscriber<T>(child, delayErrors, maxConcurrent);
        MergeProducer<T> producer = new MergeProducer<T>(subscriber);
        subscriber.producer = producer;

        child.add(subscriber);
        child.setProducer(producer);

        return subscriber;
    }
```
MergeSubscriber是OperatorMerge的内部类
```
static final class MergeSubscriber<T> extends Subscriber<Observable<? extends T>> {

    @Override
    public void onNext(Observable<? extends T> t) {
        if (t == null) {
            return;
        }
        if (t == Observable.empty()) {
            emitEmpty();
        } else
        if (t instanceof ScalarSynchronousObservable) {
            tryEmit(((ScalarSynchronousObservable<? extends T>)t).get());
        } else {
            //给上流传入的被观察者都进行一次订阅
            InnerSubscriber<T> inner = new InnerSubscriber<T>(this, uniqueId++);
            addInner(inner);
            t.unsafeSubscribe(inner);
            emit();
        }
    }
}
```
InnerSubscriber也是OperatorMerge的内部类
```
static final class InnerSubscriber<T> extends Subscriber<T> {
    @Override
    public void onNext(T t) {
        //parent就是传入的OperatorMerge对象
        //传入的事件统一交给OperatorMerge来处理
        parent.tryEmit(this, t);
    }
}}
```
tryEmit的过程就是将结果保存到队列中，然后逐一发送给下游
* 至此，flatMap操作符也结束了
