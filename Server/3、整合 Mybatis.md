# 添加 meavn 依赖

```xml
<dependency>
    <groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.30</version>
</dependency>

<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-jdbc</artifactId>
		<version>2.7.3</version>
</dependency>

<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>2.2.2</version>
</dependency>
```



# 数据库连接

```properties
spring.datasource.username=root
spring.datasource.password=xxxxxx
spring.datasource.url=jdbc:mysql://localhost:3306/blog?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```



# 新建实体类

```java
package com.devil.blog.entity;

public class Category {
    private int id;
    private int pid;
    private String name;
    private String description;
    public Category(int id, int pid, String name, String description) {
        this.id = id;
        this.pid = pid;
        this.name = name;
        this.description = description;
    }
    public Category() {
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public int getPid() {
        return pid;
    }
    public void setPid(int pid) {
        this.pid = pid;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    @Override
    public String toString() {
        return "Category [description=" + description + ", id=" + id + ", name=" + name + ", pid=" + pid + "]";
    }
}
```



# 创建 mapper 接口

```java
package com.devil.blog.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;

import com.devil.blog.entity.Category;

@Mapper
@Repository
public interface CategoryMapper {
    List<Category> queryCategories();
}
```



# 创建 mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.devil.blog.mapper.CategoryMapper">
  <select id="queryCategories" resultType="Category">
    select * from category 
  </select>
</mapper>
```



# 测试

```java
package com.devil.blog.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import com.devil.blog.entity.Category;
import com.devil.blog.mapper.CategoryMapper;

@RestController
public class CategoryController {
    @Autowired
    private CategoryMapper categoryMapper;

    @GetMapping("/categories")
    public List<Category> queCategories() {
        List<Category> categories = categoryMapper.queryCategories();
        for(Category c : categories) {
            System.out.println(c);
        }
        return categories;
    }
    
}
```

