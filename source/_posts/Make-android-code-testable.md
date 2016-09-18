---
title: 让Android代码变得可测试-JUnit + Robolectric + PowerMock + Mockito
date: 2016-09-18 21:35:34
tags:
---
自动化测试是我们常常幻想的事，有了它测试那边不需要再排期，对case的覆盖也更全面，到现在终于有机会做起来了。
对于Android来说，第一个问题是，你需要跑在Instrumentation下，还是跑在JUnit下。我们先看看在两个框架下写一个unit test是怎样的。
一个简单的instrumentation test代码如下
```
public class SimpleInstrumentationTest extends InstrumentationTestCase {

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        //do something before all tests
    }

    public void testCalculation() {
        assertTrue(1 + 1 == 2);
    }
```
而一个简单的JUnit test代码如下
```
public class SimpleJUnitTest{

    @Before
    public void setUp() throws Exception {
        super.setUp();
        //do something before every tests
    }

    @Test
    public void testCalculation() {
        assertTrue(1 + 1 == 2);
    }
```
## JUnit: simple and fast
运行Instrumentation test，你需要一个Android设备，至少是模拟器作为运行平台。对于CI来说，这要花不少的时间。由于跑在机器上，Instrumentation test拥有完整的Context及生命周期控制，还可以加载native library，你完全可以把它看做一个有点特殊的App。
相较而言，JUnit简单的多，它运行在JVM上，跑起来很快，可以测试大部分的纯Java代码逻辑。对于Context和生命周期的问题，可以使用Robolectric框架来解决，另外JUnit还支持一些Mock框架。
## Robolectric: android enviroment
平常在查看android source code的时候，会发现很多方法都只有stub，看不到真正的实现，这些实现通常会在机器内提供，所以当你在JUnit调用`android.util.Log.i(TAG,"xxx")`的时候，会提示`java.lang.NoClassDefFoundError: android/util/Log`。因为运行JUnit的JVM并没有提供android.jar。为了在JUnit下做Android test，拿到Context之类的东西，我们引入了Robolectric框架。详细的使用请见官网。
Robolectric官网：http://robolectric.org/
## PowerMock+Mockito: mock method
搞定了Android 环境之后，在实现过程中还遇到了一些问题，譬如nativeLibrary无法载入，为了覆盖到所有情况需要一些fake class。这些都需要对原有的方法进行mock。
如果只是mock一些对象，或者mock对象提供的方法，那么只用Mockito就可以满足。但事如果要mock static方法，就需要用到Powermock来实现了。这里我们讲解一些mock的方法
### 首先是原方法
```
class AES {
    public static String encrypt(String str) {
        //do aes
        return str;
    }
}
```

### mock原方法，执行原方法体内代码，并返回指定的值
```
 when(AES.encrypt(anyString())).thenReturn("aa");   //对所有调用AES.encrypt()均返回"aa"
```
### mock原方法，不执行原方法体内代码，直接返回指定值
```
 doReturn("aa").when(AES.encrypt(anyString()));   //对所有调用AES.encrypt()均返回"aa"
```
### mock原方法，对输入参数进行处理，然后传出(PowerMockito)
这里比较特殊，在声明完之后，还要紧接着调用一下原方法
```
PowerMockito.doAnswer(new Answer<Long>() {
            @Override
            public Long answer(InvocationOnMock invocation) throws Throwable {
                arg = (String) invocation.getArguments()[1]
                return arg + "aa";
            }
        }).when(AES.class);
```
### Test环境屏蔽加载native library
一般我们会在static代码块加载native library，那么屏蔽掉static 代码块就好了:)
```
@SuppressStaticInitializationFor({
    "full.class.name"
})
```