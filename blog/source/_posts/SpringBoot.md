---
title: SpringBoot
date: 2022-03-25 19:03:20
tags:
---
## properties
### application.properties
```properties
  environments.dev.url=http://ll.com
  environments.dev.name=Developer

  server.port=8080

  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  spring.datasource.url=jdbc:mysql://localhost:3306/flash_sale?
  serverTimezone=GMT&useUnicode=true&characterEncoding=utf8&useSSL=true
  # 记得更改用户名和密码为本地所使用的
  spring.datasource.username=root
  spring.datasource.password=root
  # MyBatis Mapper Config
  mybatis.mapper-locations=classpath:mappers/*.xml
```
### yml
```yml
  environments:
    dev:
      url:http://ll.com
      name:Developer
```
### 读取配置
* `@Value("${school.grade}")`
* `@ConfigurationProperties(prefix = "school")` and `@Component` on SchoolConfig.java
```Java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 描述：    读取配置类
 */
@RestController
public class ConfigController {

    @Autowired
    SchoolConfig schoolConfig;


    @GetMapping({"/gradefromconfig"})
    public String gradeclass() {
        return "年级: " + schoolConfig.grade + " 班级：" + schoolConfig.classnum;
    }
}
```

### xxApplication.java
```
@SpringBootApplication
@MapperScan("com.ll.flashsale.db.mappers")
@ComponentScan(basePackages = {"com.ll"})
```


## Annotations
* `@Mapper`

## Steps
* create project
* start application
* create a test controller
* test the test controller
* create database
* configure MyBatis
* run MyBatis generator
* add `@MapperScan` and `@ComponentScan`
* test Dao
* add front end
