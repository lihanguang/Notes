#### 链接

https://www.cnblogs.com/hunterCecil/p/5721468.html

#### 介绍

PowerMock是对Mockito的补充。

#### 基本配置

 如果你的测试用例里没有使用注解@PrepareForTest，那么可以不用加注解@RunWith(PowerMockRunner.class)，反之亦然。 

```groovy
testCompile org.powermock:powermock-module-junit4:1.7.1
testCompile org.powermock:powermock-api-mockito2:1.7.1
```

#### 基本用法

* 普通Mock：Mock参数传递的对象

```java
//测试目标代码：
public boolean callArgumentInstance(File file) {
    return file.exists();
}
 
//测试用例代码：
@Test
public void testCallArgumentInstance() {
    File file = PowerMockito.mock(File.class);
    Source underTest = new Source();
    PowerMockito.when(file.exists()).thenReturn(true);
    Assert.assertTrue(underTest.callArgumentInstance(file));
}
```

 说明：普通Mock不需要加@RunWith和@PrepareForTest注解，以下Mock需要。 

* Mock方法内部new出来的对象

```java
//测试目标代码：
public boolean callInternalInstance(String path) {
    File file = new File(path);
    return file.exists();
}
  
//测试用例代码：   
@Test
@PrepareForTest(Source.class)
public void testCallInternalInstance() throws Exception
{
    File file = PowerMockito.mock(File.class);
    Source underTest = new Source();
    PowerMockito.whenNew(File.class).withArguments("bbb").thenReturn(file);
    PowerMockito.when(file.exists()).thenReturn(true);
    Assert.assertTrue(underTest.callInternalInstance("bbb"));
}
```

* Mock普通对象的final方法

```java
//测试目标代码：
public class Source {
    public boolean callFinalMethod(SourceDepend refer) {
        return refer.isAlive();
    }
}
public class SourceDepend {
    public final boolean isAlive() {
        return false;
    }
}
  
//测试用例代码：
@Test
@PrepareForTest(SourceDepend.class)
public void testCallFinalMethod()
{
    SourceDepend depencency = PowerMockito.mock(SourceDepend.class);
    Source underTest = new Source();
    PowerMockito.when(depencency.isAlive()).thenReturn(true);
    Assert.assertTrue(underTest.callFinalMethod(depencency));
}
```

* Mock普通类的静态方法

```java
//测试目标代码：
public boolean callStaticMethod() {
      return SourceDepend.isExist();
}
public static boolean isExist()
{
    return false;
}
 
//测试用例代码：
@Test
@PrepareForTest(SourceDepend.class)
public void testCallStaticMethod() {
    Source underTest = new Source();
    PowerMockito.mockStatic(SourceDepend.class);
    PowerMockito.when(SourceDepend.isExist()).thenReturn(true);
    Assert.assertTrue(underTest.callStaticMethod());
}
```

* Mock私有方法

```java
//测试目标代码：
public boolean callPrivateMethod() {
    return isExist();
}
 
private boolean isExist() {
    return false;
}
  
//测试用例代码： 
@Test
@PrepareForTest(Source.class)
public void testCallPrivateMethod() throws Exception
{
    Source underTest = PowerMockito.mock(Source.class);
    PowerMockito.when(underTest.callPrivateMethod()).thenCallRealMethod();
    PowerMockito.when(underTest, "isExist").thenReturn(true);
    Assert.assertTrue(underTest.callPrivateMethod());
}
```

* Mock系统类的静态和final方法

```java
//测试目标代码：  
public boolean callSystemFinalMethod(String str)
{
    return str.isEmpty();
}
 
public String callSystemStaticMethod(String str)
{
    return System.getProperty(str);
}
  
//测试用例代码：
@Test
@PrepareForTest(Source.class)
public void testCallSystemStaticMethod()
{
    Source underTest = new Source();
    PowerMockito.mockStatic(System.class);
    PowerMockito.when(System.getProperty("aaa")).thenReturn("bbb");
    Assert.assertEquals("bbb", underTest.callSystemStaticMethod("aaa"));
}
```

* 验证静态方法

```java
// (1) 验证静态方法：
PowerMockito.verifyStatic();
Static.firstStaticMethod(param);
// (2) 扩展验证:
PowerMockito.verifyStatic(Mockito.times(2)); // 被调用2次
Static.thirdStaticMethod(Mockito.anyInt()); // 以任何整数值被调用
```
