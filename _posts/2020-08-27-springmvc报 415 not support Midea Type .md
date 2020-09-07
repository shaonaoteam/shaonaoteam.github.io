---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 改bug
---
##  springmvc报错
>415not support Midea Type 
>>环境：   
spring-mvc:5.0.2.RELEASE   
spring:5.0.2.RELEASE 

## 原因分析
找了一遍源码，打了个断点发现，application/json 传输的payload体，需要手动加上application的注解

但是如果直接设置三次` fastJsonHttpMessageConverter.setSupportedMediaTypes()`，就会覆盖掉其他的，选择自定义list装MideaType会报`/`之类的符号是非法字符串,然后就选择使用了`MediaType.parseMediaTypes()`方法，
源码如下：

```
public static List<MediaType> parseMediaTypes(@Nullable String mediaTypes) {
        if (!StringUtils.hasLength(mediaTypes)) {
            return Collections.emptyList();
        } else {
            String[] tokens = StringUtils.tokenizeToStringArray(mediaTypes, ",");
            List<MediaType> result = new ArrayList(tokens.length);
            String[] var3 = tokens;
            int var4 = tokens.length;

            for(int var5 = 0; var5 < var4; ++var5) {
                String token = var3[var5];
                result.add(parseMediaType(token));
            }

            return result;
        }
    }

```
他会帮你拆分符号，让Type和MideaType分开，不用你自己手动去设置

## 修改点如下：

```
@ComponentScan("xyz.zjhwork")
@EnableWebMvc
@Configuration
public class MvcConf implements WebMvcConfigurer {



    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
//        字符转换  包括解决中文乱码
        FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
        fastJsonHttpMessageConverter.setSupportedMediaTypes(MediaType.parseMediaTypes("text/html;charset=utf-8,application/json,application/json;charset=utf-8"));
        converters.add(fastJsonHttpMessageConverter);
    }

```