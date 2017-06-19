---
layout: title
title: MyBatis分页插件PageHelper
date: 2017-06-19 09:55:19
tags: [MyBatis, PageHelper]
categories: PageHelper
---

# 简介
PageHelper是一个Mybatis的分页插件，可以方便地对查询结果进行分页和排序。

# 在spring + mybatis 中使用 pagehelper

## 1. maven添加依赖
```
<!-- mybatis 分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>4.1.4</version>
</dependency>
```

## 2. 编写mybatis-config.xml
```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-config.dtd">  
<!-- 配置管理器 -->  
<configuration>
    <plugins>  
        <!-- com.github.pagehelper为PageHelper类所在包名 -->  
        <plugin interceptor="com.github.pagehelper.PageHelper">  
            <!-- 4.0.0以后版本可以不设置该参数 -->  
            <property name="dialect" value="mysql"/>  
            <!-- 该参数默认为false -->  
            <!-- 设置为true时，会将RowBounds第一个参数offset当成pageNum页码使用 -->  
            <!-- 和startPage中的pageNum效果一样-->  
            <property name="offsetAsPageNum" value="true"/>  
            <!-- 该参数默认为false -->  
            <!-- 设置为true时，使用RowBounds分页会进行count查询 -->  
            <property name="rowBoundsWithCount" value="true"/>  
            <!-- 设置为true时，如果pageSize=0或者RowBounds.limit = 0就会查询出全部的结果 -->  
            <!-- （相当于没有执行分页查询，但是返回结果仍然是Page类型）-->  
            <property name="pageSizeZero" value="true"/>  
            <!-- 3.3.0版本可用 - 分页参数合理化，默认false禁用 -->  
            <!-- 启用合理化时，如果pageNum<1会查询第一页，如果pageNum>pages会查询最后一页 -->  
            <!-- 禁用合理化时，如果pageNum<1或pageNum>pages会返回空数据 -->  
            <property name="reasonable" value="true"/>  
            <!-- 3.5.0版本可用 - 为了支持startPage(Object params)方法 -->  
            <!-- 增加了一个`params`参数来配置参数映射，用于从Map或ServletRequest中取值 -->  
            <!-- 可以配置pageNum,pageSize,count,pageSizeZero,reasonable,orderBy,不配置映射的用默认值 -->  
            <!-- 不理解该含义的前提下，不要随便复制该配置 -->  
            <property name="params" value="pageNum=start;pageSize=limit;"/>  
            <!-- 支持通过Mapper接口参数来传递分页参数 -->  
            <property name="supportMethodsArguments" value="true"/>  
            <!-- always总是返回PageInfo类型,check检查返回类型是否为PageInfo,none返回Page -->  
            <property name="returnPageInfo" value="check"/>  
        </plugin>  
    </plugins>  
</configuration>
```

## 3. 在spring-mybatis配置文件中引入上述配置
```
<!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 -->  
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:mybatis/mybatis-config.xml" />  
    <!-- 自动扫描mapping.xml文件 -->  
    <property name="mapperLocations" value="classpath:com/demo/mapping/*.xml"></property>  
</bean>
```

## 4. 业务逻辑代码
1. mapper.xml
```
<select id="getAll" resultMap="BaseResultMap">
    SELECT 
    <include refid="Base_Column_List" />
    FROM city
    WHERE 1 = 1
    <if test="pattern != null and pattern != ''">
        AND (
            name like CONCAT('%', #{pattern}, '%')
            OR
            countrycode like CONCAT('%', #{pattern}, '%')
            OR
            district like CONCAT('%', #{pattern}, '%')
            OR
            population like CONCAT('%', #{pattern}, '%')
        )
    </if>
</select>
```

2. dao
```
public List<Map<String,Object>> getAll(Map<String,Object> params);
```

3. service
```
public List<Map<String, Object>> getAll(Map<String,Object> params);
```

4. serviceImpl
```
public List<Map<String, Object>> getAll(Map<String,Object> params) {
    return dao.getAll(params);
}
```

5. controller
```
    // 设置分页及排序参数：pageNum,页数，从1开始；pageSize，页面大小，每页查询数据量，如10；orderBy，排序字段及顺序，如"name desc"
    PageHelper.startPage(pageNum, pageSize);
    PageHelper.orderBy(orderBy);

    // 业务模糊查询参数
    Map<String, Object> params = new HashMap<String, Object>();
    params.put("pattern", patternStr);
    
    // 查询
    List<Map<String, Object>> dataList = cityService.getAll(params);
    PageInfo<Map<String, Object>> pageInfo = new PageInfo<Map<String, Object>>(dataList);
    
    // 结果
    pageInfo.getTotal();// 总结果数，如4079
    pageInfo.getList();// 数据结果，本次查询返回结果，如10个
```
6. ajax请求
```
"ajax": {
    "url": "city/getall",
    "type": "POST",
    "data": {
        pageNum: 1;		//查询第一页
        pageSize: 10;	//查询10条记录
        orderColumn: "name";	// 后台的orderBy通过这里的orderColumn + orderDir拼接而成。
        orderDir: "desc";	//但是为了防止sql注入及非法参数，最好后台增加方法判断参数合法性，并返回合法值。
        pattern: "tenny";	//业务相关参数，模糊查询
    }
}
```

## 查看结果
通过日志可以看到，插件对sql做了处理：1.先查询一条总数，2.在原sql上加入分页条件进行查询。
```
2017-06-19 11:15:38,569 DEBUG [com.demo.dao.ICityDao.getAll_COUNT] - ==>  Preparing: SELECT count(0) FROM city WHERE 1 = 1 
2017-06-19 11:15:38,601 DEBUG [com.demo.dao.ICityDao.getAll_COUNT] - ==> Parameters: 
2017-06-19 11:15:38,841 DEBUG [com.demo.dao.ICityDao.getAll_COUNT] - <==      Total: 1
2017-06-19 11:15:38,850 DEBUG [com.demo.dao.ICityDao.getAll] - ==>  Preparing: SELECT id, name, countrycode, district, population FROM city WHERE 1 = 1 order by id asc limit ?,? 
2017-06-19 11:15:38,850 DEBUG [com.demo.dao.ICityDao.getAll] - ==> Parameters: 0(Integer), 10(Integer)
2017-06-19 11:15:38,854 DEBUG [com.demo.dao.ICityDao.getAll] - <==      Total: 10
```

pageInfo封装了查询条件及结果集：
```
PageInfo{pageNum=1, pageSize=10, size=10, startRow=1, endRow=10, total=4079, pages=408, list=Page{count=true, pageNum=1, pageSize=10, startRow=0, endRow=10, total=4079, pages=408, countSignal=false, orderBy='id asc', orderByOnly=false, reasonable=true, pageSizeZero=true}, firstPage=1, prePage=0, nextPage=2, lastPage=8, isFirstPage=true, isLastPage=false, hasPreviousPage=false, hasNextPage=true, navigatePages=8, navigatepageNums=[1, 2, 3, 4, 5, 6, 7, 8]}
```