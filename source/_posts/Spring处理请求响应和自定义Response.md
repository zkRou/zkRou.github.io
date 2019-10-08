---
title: Spring处理请求响应和自定义Response
author: Kairou Zeng
date: 2019/06/07
tage:
	- HTTP
	- Spring
categories:
	- Spring
---
# 为什么写

手头接到一个项目，整个项目接口返回的格式其实都基本一致，因此可以自定义`ResponseBody`来统一控制层的输出。

# Http请求和响应

Http请求和响应报文本质上都是一串字符串，当请求报文来到java世界里，它会被封装成为一个`ServletInputStream`的输入流，供我们读取报文。响应报文则是通过一个`ServletOutputStream`的输出流，来输出响应报文。在`SpringMVC`中，`HttpMessageConverter`机制将输入流/输出流转为java对象。

- HttpMessageConverter

```java
package org.springframework.http.converter;

import java.io.IOException;
import java.util.List;

import org.springframework.http.HttpInputMessage;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.MediaType;
import org.springframework.lang.Nullable;

/**
 * Strategy interface that specifies a converter that can convert from and to HTTP requests and responses.
 *
 * @author Arjen Poutsma
 * @author Juergen Hoeller
 * @since 3.0
 * @param <T> the converted object type
 */
public interface HttpMessageConverter<T> {

	/**
	 * Indicates whether the given class can be read by this converter.
	 * @param clazz the class to test for readability
	 * @param mediaType the media type to read (can be {@code null} if not specified);
	 * typically the value of a {@code Content-Type} header.
	 * @return {@code true} if readable; {@code false} otherwise
	 */
     //判断该转换器是否能将请求内容转换为Java对象
	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

	/**
	 * Indicates whether the given class can be written by this converter.
	 * @param clazz the class to test for writability
	 * @param mediaType the media type to write (can be {@code null} if not specified);
	 * typically the value of an {@code Accept} header.
	 * @return {@code true} if writable; {@code false} otherwise
	 */
     //判断该转换器是否可以将Java对象转换成返回内容
	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

	/**
	 * Return the list of {@link MediaType} objects supported by this converter.
	 * @return the list of supported media types
	 */
     //获得该转换器支持的MediaType类型
	List<MediaType> getSupportedMediaTypes();

	/**
	 * Read an object of the given type from the given input message, and returns it.
	 * @param clazz the type of object to return. This type must have previously been passed to the
	 * {@link #canRead canRead} method of this interface, which must have returned {@code true}.
	 * @param inputMessage the HTTP input message to read from
	 * @return the converted object
	 * @throws IOException in case of I/O errors
	 * @throws HttpMessageNotReadableException in case of conversion errors
	 */
    //读取请求内容并转换成Java对象
	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	/**
	 * Write an given object to the given output message.
	 * @param t the object to write to the output message. The type of this object must have previously been
	 * passed to the {@link #canWrite canWrite} method of this interface, which must have returned {@code true}.
	 * @param contentType the content type to use when writing. May be {@code null} to indicate that the
	 * default content type of the converter must be used. If not {@code null}, this media type must have
	 * previously been passed to the {@link #canWrite canWrite} method of this interface, which must have
	 * returned {@code true}.
	 * @param outputMessage the message to write to
	 * @throws IOException in case of I/O errors
	 * @throws HttpMessageNotWritableException in case of conversion errors
	 */
    //将Java对象转换后写入返回内容
	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}
```

`HttpMessageConverter`对消息转换器最高层次的接口抽象。该类中有成对的`canRead()`、`read()`和`canWrite()`、`write()`方法，`MediaType`是对请求的`Media Type`属性的封装。

- HttpInputMessage

```java
/**
 * Represents an HTTP input message, consisting of {@linkplain #getHeaders() headers}
 * and a readable {@linkplain #getBody() body}.
 *
 * <p>Typically implemented by an HTTP request handle on the server side,
 * or an HTTP response handle on the client side.
 *
 * @author Arjen Poutsma
 * @since 3.0
 */
public interface HttpInputMessage extends HttpMessage {

	/**
	 * Return the body of the message as an input stream.
	 * @return the input stream body (never {@code null})
	 * @throws IOException in case of I/O errors
	 */
	InputStream getBody() throws IOException;

}
```

`HttpInputMessage`这个类是`SpringMVC`内部对一次Http请求报文的抽象，在`HttpMessageConverter`的`read()`方法中，有一个`HttpInputMessage`的形参，它正是`SpringMVC`的消息转换器所作用的受体“请求消息”的内部抽象，消息转换器从"请求消息"中按照规则提取信息，转换为方法形参中声明的对象。

- HttpOutputMessage

```java
package org.springframework.http;

import java.io.IOException;
import java.io.OutputStream;

/**
 * Represents an HTTP output message, consisting of {@linkplain #getHeaders() headers}
 * and a writable {@linkplain #getBody() body}.
 *
 * <p>Typically implemented by an HTTP request handle on the client side,
 * or an HTTP response handle on the server side.
 *
 * @author Arjen Poutsma
 * @since 3.0
 */
public interface HttpOutputMessage extends HttpMessage {

	/**
	 * Return the body of the message as an output stream.
	 * @return the output stream body (never {@code null})
	 * @throws IOException in case of I/O errors
	 */
	OutputStream getBody() throws IOException;

}
```

`HttpOutputMessage`这个类是`SpringMVC`内部对一次Http响应报文的抽象，在`HttpMessageConverter`的`write()`方法中，有一个`HttpOutputMessage`的形参，它正是`SpringMVC`的消息转换器所作用的受体“响应消息”的内容抽象，消息转换器将“响应消息”按照一定的规则写到响应报文中。

# 请求、响应过程

```java
@GetMapping("/string")
@ResponseBody
public String readString(@RequestBody String string){
    return "Read string '" + string + "'";
}
```

- 请求

    请求报文 -> HttpInputMessage -> HttpMessageConverter -> java对象 -> SpringMVC

    1. 进入`readString()`方法前，根据`@RequestBody`注解选择适当的`HttpMessageConverter`实现类将请求参数解析到string变量中，具体来说是使用了`StringHttpMessageConverter`类，它的`canRead()`方法返回true;
    2. 它的`read()`方法会从请求中读出请求参数，绑定到`readString()`方法的string变量中。

- 响应

    SpringMVC -> java对象 -> HttpMessageConverter -> HttpOutputMessage -> 响应报文

    1. 当`readString()`执行后，由于返回值标识了`@ResponseBody`，`SpringMVC`将使用`StringHttpMessageConverter`的`write()`方法，将结果作为String值写入响应报文中，此时它的`canWrite()`方法返回true。 

# RequestResponseBodyMethodProcessor

这个类同时实现了`HandlerMethodArgumentResolver`(将请求报文绑定到处理方法形参的策略接口)和`HandlerMethodReturnValueHandler`(对处理方法返回值进行处理的策略接口)两个接口。

`HandlerMethodArgumentResolver`:

```java
package org.springframework.web.method.support;

import org.springframework.core.MethodParameter;
import org.springframework.lang.Nullable;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;

/**
 * Strategy interface for resolving method parameters into argument values in
 * the context of a given request.
 *
 * @author Arjen Poutsma
 * @since 3.1
 * @see HandlerMethodReturnValueHandler
 */
public interface HandlerMethodArgumentResolver {

	/**
	 * Whether the given {@linkplain MethodParameter method parameter} is
	 * supported by this resolver.
	 * @param parameter the method parameter to check
	 * @return {@code true} if this resolver supports the supplied parameter;
	 * {@code false} otherwise
	 */
	boolean supportsParameter(MethodParameter parameter);

	/**
	 * Resolves a method parameter into an argument value from a given request.
	 * A {@link ModelAndViewContainer} provides access to the model for the
	 * request. A {@link WebDataBinderFactory} provides a way to create
	 * a {@link WebDataBinder} instance when needed for data binding and
	 * type conversion purposes.
	 * @param parameter the method parameter to resolve. This parameter must
	 * have previously been passed to {@link #supportsParameter} which must
	 * have returned {@code true}.
	 * @param mavContainer the ModelAndViewContainer for the current request
	 * @param webRequest the current request
	 * @param binderFactory a factory for creating {@link WebDataBinder} instances
	 * @return the resolved argument value, or {@code null} if not resolvable
	 * @throws Exception in case of errors with the preparation of argument values
	 */
	@Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```

对`HandlerMethodArgumentResolver`接口的实现：

```java
@Override
public boolean supportsParameter(MethodParameter parameter) {
    return parameter.hasParameterAnnotation(RequestBody.class);
}

@Override
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
        NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

    parameter = parameter.nestedIfOptional();
    Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
    String name = Conventions.getVariableNameForParameter(parameter);

    if (binderFactory != null) {
        WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
        if (arg != null) {
            validateIfApplicable(binder, parameter);
            if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
                throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
            }
        }
        if (mavContainer != null) {
            mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
        }
    }

    return adaptArgumentIfNecessary(arg, parameter);
}
```

`HandlerMethodReturnValueHandler`:

```java
package org.springframework.web.method.support;

import org.springframework.core.MethodParameter;
import org.springframework.lang.Nullable;
import org.springframework.web.context.request.NativeWebRequest;

/**
 * Strategy interface to handle the value returned from the invocation of a
 * handler method .
 *
 * @author Arjen Poutsma
 * @since 3.1
 * @see HandlerMethodArgumentResolver
 */
public interface HandlerMethodReturnValueHandler {

	/**
	 * Whether the given {@linkplain MethodParameter method return type} is
	 * supported by this handler.
	 * @param returnType the method return type to check
	 * @return {@code true} if this handler supports the supplied return type;
	 * {@code false} otherwise
	 */
	boolean supportsReturnType(MethodParameter returnType);

	/**
	 * Handle the given return value by adding attributes to the model and
	 * setting a view or setting the
	 * {@link ModelAndViewContainer#setRequestHandled} flag to {@code true}
	 * to indicate the response has been handled directly.
	 * @param returnValue the value returned from the handler method
	 * @param returnType the type of the return value. This type must have
	 * previously been passed to {@link #supportsReturnType} which must
	 * have returned {@code true}.
	 * @param mavContainer the ModelAndViewContainer for the current request
	 * @param webRequest the current request
	 * @throws Exception if the return value handling results in an error
	 */
	void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception;

}
```

对`HandlerMethodReturnValueHandler`的接口实现：

```java
@Override
public boolean supportsReturnType(MethodParameter returnType) {
    return (AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ResponseBody.class) ||
            returnType.hasMethodAnnotation(ResponseBody.class));
}

@Override
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
        ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
        throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

    mavContainer.setRequestHandled(true);
    ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
    ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

    // Try even with null return value. ResponseBodyAdvice could get involved.
    writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
}
```

# 自定义`@Response`实现

1. 定义一个注解类

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 为了更方便返回统一的JSON格式而自定义的注解
 * 作用类似于{@link org.springframework.web.bind.annotation.ResponseBody}
 *
 * @author Kairou Zeng
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ResultResponseBody {
}
```

2. 定义一个类，继承`RequestResponseBodyProcessor`，并重写`supportsReturnType(MethodParameter returnType)`方法和`handleReturnValue`方法

```java
import com.szfgw.IntelligentSite.bean.vo.ResponseVO;
import org.springframework.core.MethodParameter;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.HttpMessageNotWritableException;
import org.springframework.web.HttpMediaTypeNotAcceptableException;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.ModelAndViewContainer;
import org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor;

import java.io.IOException;
import java.util.List;

/**
 * 结合{@link ResultResponseBody}包装处理返回结果
 *
 * @author Kairou Zeng
 */
public class ResultResponseHandlerMethodProcessor extends RequestResponseBodyMethodProcessor {

    public ResultResponseHandlerMethodProcessor(List<HttpMessageConverter<?>> converters) {
        super(converters);
    }


    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        return returnType.getMethodAnnotation(ResultResponseBody.class) != null;
    }

    @Override
    public void handleReturnValue(Object returnValue,
                                  MethodParameter returnType,
                                  ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {
        super.handleReturnValue(new ResponseVO(returnValue), returnType, mavContainer, webRequest);
    }
}
```

3. 实现接口`WebMvcConfigurer`的`addReturnValueHandlers`方法

```java
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.szfgw.IntelligentSite.utils.ResultResponseHandlerMethodProcessor;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.web.method.support.HandlerMethodReturnValueHandler;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.Collections;
import java.util.List;

/**
 * @author Kairou Zeng
 */
 @Configuration
public class WebMvcConfig implements WebMvcConfigurer {
     
    private final MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter;

    public WebMvcConfig(MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter) {
        this.mappingJackson2HttpMessageConverter = mappingJackson2HttpMessageConverter;
    }

    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
        ObjectMapper objectMapper = mappingJackson2HttpMessageConverter.getObjectMapper();
        objectMapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
        objectMapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
        handlers.add(new ResultResponseHandlerMethodProcessor(Collections.singletonList(mappingJackson2HttpMessageConverter)));
    }
}
```