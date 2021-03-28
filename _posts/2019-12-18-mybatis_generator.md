---
layout: default
title: mybatis自动生成代码
#excerpt: 

---

# 1.依赖jar

mybatis-generator-core-1.3.2.jar

mysql-connector-java-5.1.38.jar



# 2.数据源、代码位置等配置

generatorConfig.xml，注意，要在generatorConfig.xml同一级目录，生成 src/main和src/resources目录

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--数据库驱动-->
    <classPathEntry location="mysql-connector-java-5.1.38.jar"/>

    <context id="DB2Tables" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://127.0.0.1:3306/test" userId="root" password="">
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--生成Model类存放位置-->
        <javaModelGenerator targetPackage="com.frank.model" targetProject="src/main">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!--生成xml文件存放位置-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--生成Dao接口存放位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.frank.dao" targetProject="src/main">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <!--生成对应表及类名-->
        <table tableName="student" domainObjectName="student" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
    </context>
</generatorConfiguration>
```


# 3.点击生成

戳我生成mybatis代码.bat，注意要在 generatorConfig.xml 中写清要生成文件，对应数据库的表名字

```bash
java -jar mybatis-generator-core-1.3.2.jar -configfile generatorConfig.xml -overwrite
```

