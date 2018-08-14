#### 介绍

Espresso是一个Google官方提供的Android应用UI自动化测试框架。Google希望，当Android的开发者利用Espresso写完测试用例后，能一边看着测试用例自动执行，一边享受一杯香醇Espresso(浓咖啡)。 Espresso有三个特点：

* 第一个收录在Android Testing Supporting Library底下的测试框架
* 模拟用户的操作
* 自动等待，直到UI线程Idle，才会执行测试代码

#### 配置

* 修改APP的build.gradle

```
defaultConfig {
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}
dependencies {
    // Espresso 相关的引用
    androidTestCompile('com.android.support.test.espresso:espresso-core:3.0.2') {
        exclude group: 'com.android.support'
    }
    androidTestCompile('com.android.support.test:runner:1.0.2'){
        exclude group: 'com.android.support'
    }
    androidTestCompile('com.android.support.test:rules:1.0.2'){
        exclude group: 'com.android.support'
    }
    androidTestCompile('com.android.support.test.espresso:espresso-contrib:3.0.2') {
        exclude group: 'com.android.support'
    }
    // Junit
    androidTestCompile "junit:junit:4.12"
    androidTestCompile "org.mockito:mockito-core:1.10.19"
}
```

#### 注意事项

* Espresso是运行在设备上的，所以需要把用例都放在AndroidTest目录下而不是Test目录下
* Espresso的依赖会导入一些support包，所以需要解决一下依赖冲突

#### 用法

* 配置类

```java
// 1、类名前添加注解
@RunWith(AndroidJUnit4.class)
public class ChooseLanguageActivityTest {

    // 添加Rule变量，需要用注解
    @Rule
    public ActivityTestRule<Activity> mTasksActivityTestRule =
            new ActivityTestRule<Activity>(Activity.class);
    
}
```

* 主要用法分为三步
  * 定位View
  * 操作View
  * 检验View

```java
// 概括为代码为
onView(ViewMatcher)
  .perform(ViewAction)
  .check(ViewAssertion);
```

#### Sample

```java
/**
 * Created by Lihanguang on 2018/8/13.
 */
@RunWith(AndroidJUnit4.class)
@LargeTest
public class ChooseLanguageActivityTest {

    @Rule
    public ActivityTestRule<ChooseLanguageActivity> mTasksActivityTestRule =
            new ActivityTestRule<ChooseLanguageActivity>(ChooseLanguageActivity.class);

    @Test
    public void testOnCreate() {
        onView(withId(R.id.rv_language)).perform(RecyclerViewActions.actionOnItemAtPosition(1, click()));

        onView(new RecyclerViewMatcher(R.id.rv_language).atPositionOnView(1, R.id.iv_choose)).check(matches(isDisplayed()));

        onView(withId(R.id.tv_confirm)).check(matches(isEnabled()));
    }

    @Test
    public void testClick() {
        onView(withId(R.id.rv_language)).perform(RecyclerViewActions.actionOnItemAtPosition(1, click()));

        onView(new RecyclerViewMatcher(R.id.rv_language).atPositionOnView(1, R.id.iv_choose)).check(matches(isDisplayed()));

        onView(withId(R.id.tv_confirm)).check(matches(isEnabled()));

        onView(withId(R.id.rv_language)).perform(RecyclerViewActions.actionOnItemAtPosition(2, click()));

        onView(new RecyclerViewMatcher(R.id.rv_language).atPositionOnView(1, R.id.iv_choose)).check(matches(not(isDisplayed())));
    }

    @Test
    public void testSkip() {
        onView(withId(R.id.tv_skip)).perform(click());

        onView(withId(R.id.tab_pager)).check(matches(isDisplayed()));
    }

    @Test
    public void testOK() {
        onView(withId(R.id.rv_language)).perform(RecyclerViewActions.actionOnItemAtPosition(1, click()));

        onView(new RecyclerViewMatcher(R.id.rv_language).atPositionOnView(1, R.id.iv_choose)).check(matches(isDisplayed()));

        onView(withId(R.id.tv_confirm)).check(matches(isEnabled()));

        onView(withId(R.id.tv_confirm)).perform(click());

        onView(withId(R.id.tab_pager)).check(matches(isDisplayed()));
    }
}
```

