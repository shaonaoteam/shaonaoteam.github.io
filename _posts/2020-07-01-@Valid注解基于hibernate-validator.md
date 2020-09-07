---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
##  `hibernate-validator`
是基于`javax.validation`的数据合法性检测包，用于检测常用类的极限值、空值、边界值等的数据合法性校验
###  引入

```
<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>6.1.5.Final</version>
</dependency>

```
###  `@valid`   编写`controller`
用于校验`controller`传输对象`DTO`内的参数合法性注解，通过反射获取`controller`的类加载器，判断`method`内是否使用到`@valid`注解，从而获取响应对象的参数进行合法性校验；


```
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

/**
 * @Valid注解测试接口
 */
@RestController
//Swagger tags
@Api(tags = "@Valid注解测试接口")
public class ValidTestController {
	//Swagger 方法描述
    @ApiOperation(value = "validGet测试", notes = "Get测试")
    @RequestMapping(value = "/testValid",method = RequestMethod.GET)
    public R<ValidDTO> testValid(@Valid ValidDTO validDTO){
        return R.ok(validDTO);
    }
}

```
###  `@NotBlank` `@NotNull` `@Max` `@mail`等   编写`DTO`
此类注解是用于描述数据传输对象`DTO`的成员变量，用于该对象在传输时并且使用到该对象的和使用了`@valid`注解`Controller` 的参数合法性检验

```
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import javax.validation.constraints.Email;
import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

@Data
//swaggerModel描述
@ApiModel("valid数据格式检测注解测试")
public class ValidDTO {

    @NotNull(message = "id不能为空")
    private Integer id;

    /***    swaggerModel成员变量描述
     *     @ApiModelProperty注解如果注释类型是integer 等基本类型包装类 需要赋初值
      */
    @ApiModelProperty("ValidDTOName")
    @NotBlank(message = "name不能为空")
    private String name;

    @ApiModelProperty("ValidDTOEmail")
    @Email(message = "邮箱格式不对")
    @NotBlank(message = "邮箱不能为空")
    private String email;

    @Max(value = 10,message = "money长度超长")
    private Integer money;

    /**
     * 不写message的默认提示:cn.temp.tempbeans.conf.Exception - 最大不能超过2
     */
    @Max(value = 2)
    private Integer age;
}

```
###  编写自定义异常处理类  
#### 1、该处的`@ControllerAdvice`注解是用于处理`controller`的全局异常，当然该类还有其他的作用，例如全局参数配置、全局数据预处理之类的
#### 2、`@ExceptionHandler(org.springframework.validation.BindException.class)`这是对`validation.BindException`异常进行适配处理，以下方法则是对我们参数非法的自定义格式处理，为了遵循返回数据格式的一致性，还是采用`mybatis-plus`的`R`数据返回类做返回处理

```
import com.baomidou.mybatisplus.extension.api.R;
import com.baomidou.mybatisplus.extension.enums.ApiErrorCode;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.Objects;

@Slf4j
@ControllerAdvice
public class Exception {

    private final static String EXCEPTION_MSG_KEY = "Exception message : ";

    @ResponseBody
    @ExceptionHandler(org.springframework.validation.BindException.class)
    public R<String> handleValidException(BindException e){
        //日志记录错误信息
        log.error(Objects.requireNonNull(e.getBindingResult().getFieldError()).getDefaultMessage());
        //将错误信息返回给前台
        return R.restResult(Objects.requireNonNull(e.getBindingResult().getFieldError()).getDefaultMessage(), ApiErrorCode.FAILED);
    }
}

```



## 此处还用到了`R.ok` `R.fail`等封装返回对象
#### 该类是`mybatis-plus`的数据返回对象，用于格式化最终返回参数集，类似于在我的以往项目中封装的项目总体的返回集传输对象`ResModel`,数据返回格式:

```  
//R.restResult(Object,IErrorCode)
 {
  "code": -1,
  "data	": "最大不能超过2",
  "msg": "操作失败"
}
//R.ok(Object)
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "username": "zjh",
      "password": "adcd7048512e64b48da55b027577886ee5a36350"
    }
  ],
  "msg": "执行成功"
}

```
![20200629](http://img.zjhwork.xyz/checkParam.png)
