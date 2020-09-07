---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
## 注意事项
如果ssm项目中使用的fastJson包进行@ResponseBody包的导入，那么，一定记得把前端项目中的所有的JSON.parse给放开，因为Jackson会自动把返回格式给你解析出来，再加一次会报语法错误
## 实现
pom.xml

```
<dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.11.0</version>
        </dependency>

```
Swagger2Config.java

```
package xyz.zjhwork.springApplicationStarter.mvcConf;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
public class Swagger2Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("xyz.zjhwork.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Chester Blog swagger")
                .description("everything will be better!")
                .termsOfServiceUrl("http://zjhxyy.cn")
                .version("2.0")
                .build();
    }
}


```
使用

```
@Api(tags = "Approve Service Interfaces")
@ApiOperation(value = "查询是否点赞接口", notes = "查询当前文章是否被当前用户赞过，权限控制")

```