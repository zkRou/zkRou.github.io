---
title: 写UT遇到的坑和解决方法
author: Kairou Zeng
date: 2018-04-16
tags: 单元测试
---

----
**问题场景：** 发送短信验证码的接口，验证码是一个由Util工具类生成的随机数，业务层需要验证结果

**解决方案：** 使用PowerMock

> PowerMock扩展了EasyMock和Mockito框架，增加对static、final、私有方法的支持。

`PowerMock`有两个重要的注解：

- `@RunWith(PowerMockRunner.class)`
- `@PrepareForTest({YourClassEgStaticMethod.class})`

如果你的测试用例没有使用注解`@PrepareForTest`,那么可以不用加注解`@RunWith(PowerMockRunner.class)`,反之亦然。当你需要使用PowerMock强大功能(Mock静态、final、私有方法等)的时候，就需要加注解`@PrepareForTest`.

当某个测试方法被注解`@PrepareForTest`标注以后，在运行测试用例时，会创建一个新的`org.powermock.core.classloader.MockClassLoader`实例，然后加载该测试用例使用到的类(系统类除外)

`PowerMock`会根据你的mock要求，去修改写在注解`@PrepareForTest`里的class文件(当前测试类会自动加入注解中),以满足特殊的mock需求.例如:去除final方法的final标识,在静态方法的最前面加入自己的虚拟实现等。

如果需要mock的是系统类的final方法和静态方法,`PowerMock`不会直接修改系统类的class文件，而是修改调用系统类的class文件,以满足mock需求.

- 验证静态方法

PowerMockito.verifyStatic(); 

```java
PowerMockito.verifyStatic(Mockito.times(2)); //被调用两次
```

Static.firstStaticMethod(param);

```java
tatic.thirdStaticMethod(Mockito.anyInt()); // 以任何整数值被调用
```

**实际解决代码：**
```java
@SpringBootTest
@RunWith(PowerMockRunner.class)
@PowerMockRunnerDelegate(SpringRunner.class)
@AutoConfigureMockMvc
@PowerMockIgnore({"javax.net.*", "javax.security.*"})
public class SmsVerificationCodeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Mock
    private ThirdClient thirdClient;

    @MockBean
    private SmsVerificationCodeDao smsVerificationCodeDao;

    /**
     * 发送短信验证码
     */
    @Test
    @PrepareForTest(RandomUtils.class) 
    //当mock静态方法、final方法时，必须加注解`@PrepareForTest`和`@RunWith`。注解`@PrepareForTest`里写的类是静态方法/final方法所在的类
    public void sendVerificationCodeSms() throws Exception {
        String code = "000000";
        String telephone = "136********";

        Response response = new Response();
        response.setErrcode(0);

        PowerMockito.mockStatic(RandomUtils.class);
        PowerMockito.when(RandomUtils.getCodeChars()).thenReturn(code.toCharArray());
        BDDMockito.given(thirdClient.sendSmsVcode(telephone, code)).willReturn(kdCloudResponse);

        mockMvc.perform(post("/send/vcode")
                .param("telephone", telephone))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("errcode").value(0))
                .andExpect(MockMvcResultMatchers.jsonPath("description").value("ok"))
                .andExpect(MockMvcResultMatchers.jsonPath("data").isEmpty());

        BDDMockito.verify(smsVerificationCodeDao).insert(any(), anyString());
    }
}
```

----
**问题场景：** 当遇到方法返回值为void,但又需要对返回值进行值相等判断时

**解决方案：** 使用`Mockito.doAnswer`

**实际解决代码：**
```java
@Test
public void loginWithNewUser() throws Exception {
    String username = "test@qq.com";
    String password = "123456";

    Mockito.doAnswer(invocation -> {
        User user = (User) invocation.getArguments()[0];
        user.setId("10000");
        return user;
    }).when(userDao).save(any(User.class));

    mockMvc.perform(post("/user/login")
            .param("username", username)
            .param("password", password))
            .andExpect(MockMvcResultMatchers.status().isOk())
            .andExpect(MockMvcResultMatchers.jsonPath("errcode").value(0))
            .andExpect(MockMvcResultMatchers.jsonPath("data.id").value("10000"));

    BDDMockito.verify(kdUserDao).save(any());
}
```