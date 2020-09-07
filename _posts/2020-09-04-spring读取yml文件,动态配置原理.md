---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
## 用spring读取yml配置文件
>pom.xml

```
 <dependency>
            <groupId>org.yaml</groupId>
            <artifactId>snakeyaml</artifactId>
            <version>1.25</version>
        </dependency>


```
>读取

```
   YamlPropertiesFactoryBean yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
   yamlPropertiesFactoryBean.setResources(new ClassPathResource("application.yml"));
   Properties properties = yamlPropertiesFactoryBean.getObject();
   properties.get("tomcat.context-path")

```
## 巧妙利用yml实现动态配置原理
>xxx.yml

```
tomcat:
  current-system: linux
config:
  windows:
    context-path: /
    doc-base: C://test/
    port: 9001
  linux:
    context-path: /
    doc-base: /usr/local/project/tomcatDoc
    port: 80

```
>使用
tomcatStarter.java

```
import lombok.Builder;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.startup.Tomcat;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.core.io.ClassPathResource;

import java.io.IOException;
import java.util.Properties;
@Builder
public class BootStarter {
    private static YamlPropertiesFactoryBean yamlPropertiesFactoryBean ;
    private static Properties properties;
    private static String currentSystem;
    static{
        yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
        yamlPropertiesFactoryBean.setResources(new ClassPathResource("application.yml"));
        properties = yamlPropertiesFactoryBean.getObject();
        currentSystem = properties.getProperty("tomcat.current-system");
    }
    public  void run(){
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.parseInt(properties.getProperty("config."+currentSystem+".port")));
        // 标识tomcat启动为webapp
        tomcat.addWebapp(properties.get("config."+currentSystem+".context-path").toString(),properties.get("config."+currentSystem+".doc-base").toString());
        try {
//            tomcat启动
            tomcat.start();
//            tomcat监听用户接入
            tomcat.getServer().await();
        } catch (LifecycleException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        BootStarter.builder().build().run();
    }
}


```