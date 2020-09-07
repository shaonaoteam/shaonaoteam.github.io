---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
# springboot整合mybatis-plus的动态crud操作，整合Swagger


## pojo

```

import lombok.Data;

import java.io.Serializable;

@Data
public class Msg  implements Serializable {
    private Integer id;
    private String content;
}


```


## daoMapper

```

import cn.temp.tempbeans.pojo.Msg;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.springframework.stereotype.Repository;

@Repository
public interface MsgMapper extends BaseMapper<Msg> {
}


```

## pom.xml

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.3.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.11</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.6.1</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.6.1</version>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>RELEASE</version>
            <scope>compile</scope>
        </dependency>

```


## controller
### 注
`mybatisplusBaseMapper`中的`Insert(User)`方法，
如果数据库字段主键id是递增的，直接填入0，他就自增的新添数据,如果需要`指定id`的填入,建议采用` UUID.randomUUID().toString();`的方式防止主键重复异常发生

```
import cn.temp.tempbeans.dao.UserMapper;
import cn.temp.tempbeans.pojo.User;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Api("测试接口类")
@RestController
@RequestMapping("/user")
public class TestController {
    @Autowired
    private UserMapper userMapper;

    @ApiOperation(value = "获取用户列表", notes = "用户列表")
    @RequestMapping(value = "/user", method = RequestMethod.GET)
//    @ApiImplicitParam(name = "user", value = "用户信息", required = false, dataType = "User")
    public List<User> getUser(User user) {
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        if(user.getUsername()!=null){
            userQueryWrapper.like("username", user.getUsername());
        }
        if(user.getId()!=null){
            userQueryWrapper.like("id", user.getId());
        }
        if(user.getPassword()!=null){
            userQueryWrapper.like("password", user.getPassword());
        }
        return userMapper.selectList(userQueryWrapper);

    }

    @ApiOperation(value = "新增用户", notes = "添加用户")
    @Transactional
    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public boolean addUser(@RequestBody User user) {
        return  userMapper.insert(user)==1;
    }

    @ApiOperation(value = "删除用户", notes = "删除用户")
    @Transactional
    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
    public boolean deleteUser( User user) {
        UpdateWrapper<User> userUpdateWrapper = new UpdateWrapper<>();
        if(user.getUsername()!=null){
            userUpdateWrapper.eq("username", user.getUsername());
        }
        if(user.getId()!=null){
            userUpdateWrapper.eq("id", user.getId());
        }
        if(user.getPassword()!=null){
            userUpdateWrapper.eq("password", user.getPassword());
        }
        return  userMapper.delete(userUpdateWrapper)!=0;
    }

    @ApiOperation(value = "更新用户", notes = "更新用户")
    @Transactional
    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    public boolean updateUser(@RequestBody User user) {
        return  userMapper.updateById(user)==1;
    }
}

```
## http://127.0.0.1:8080/swagger-ui.html
![1192254364](http://img.zjhwork.xyz/mybatis-plus.png)

