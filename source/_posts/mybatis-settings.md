---
title: MyBatis设置
date: 2017-08-02 10:26:10
tags: MyBatis
categories: Database
---
MyBatis的XML文件主要有以下配置内容：  

```Java
<configureation>
	<properties/> <!--属性-->
	<settings/> <!--设置-->
	<typeHandlers/> <!--类型处理器-->
	<objectFactory/> <!--对象工厂-->
	<plugins/> <!--插件-->
	<environments> <!--配置环境-->
		<environment> <!--环境变量-->
			<transactionManager/> <!--事务管理器-->
			<dataSource/>  <!--数据源-->
		<environment/>
	<environments/>
	<databaseIdProvider/> <!--数据库厂商标志-->
	<mappers/> <!--映射器-->
</configuration>
```  

接下来主要讲几个重要的参数。  
#### 1. 别名  
别名是一个指代的名称，我们在遇到的类全限定名过长时，使用别名可以简化。  
除去一些系统自定义的别名外（系统定义的别名包括了一些常见的类型别名），我们可以自定义别名。  
自定义别名方式如下：  
```Java
<typeAliases> 
	<typeAlizs alias="role" type="com.xx.yy.Role">
</typeAliases>
```  
这是最简单的自定义别名方式，当然在POJO类很多时，我们也可以根据包名来定义别名（即包内的POJO类会使用特定的别名）  
```Java
<typeAliases>
	<package name="com.xx.yy.zz">
</typeAliases>
```  
当字节使用上面方式时，MyBatis会自动扫描包，然后包内的POJO类的别名变成POJO类首字母小写的方式，当然也可以自定义包内别名，只需加上@Alias注解即可，举例如下：  
```Java
@Alias("role")
public class Role {
//...
}
```  
#### 2. typeHandler类型处理器  
MyBatis在预处理语句（PreparedStatement）中设置一个参数时，或者从结果集（ResultSet）中取出一个值时，都会用注册了的typeHandler进行处理。  
typeHander也分为系统定义和用户自定义两种。
扩展见[mybatis中TypeHandles使用与扩展](http://cczakai.iteye.com/blog/1265417)    
系统定义的是一些常见的JDBC类型，例如：  
```  
BooleanTypeHandler -- java.lang.Boolean, boolean -- 数据库兼容的BOOLEAN  
ByteTypeHandler  -- java.lang.Byte, byte -- 数据库兼容的NUMERIC或BYTE  
...
```  

##### 1. 自定义typeHandler举例  
- 自定义typeHandler实现   

```Java  
package com.learn.chapter3.typeHandler;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;
import org.apache.ibatis.type.MappedTypes;
import org.apache.ibatis.type.TypeHandler;
import org.apache.log4j.Logger;

@MappedTypes({String.class}) //定义JavaType类型
@MappedJdbcTypes({JdbcType.VARCHAR}) //定义JDBC类型
public class MyStringTypeHandler implements TypeHandler<String>{  
	private Logger log = Logger.getLogger(MyStringTypeHandler.class);
	@Override
	public String getResult(ResultSet rs, String colName) throws SQLException {
		log.info("使用我的TypeHandler, ResultSet列明获取字符串");
		return rs.getString(colName);
	}

	@Override
	public String getResult(ResultSet rs, int index) throws SQLException {
		log.info("使用我的TypeHandler, ResultSet下标获取字符串");
		return rs.getString(index);
	}

	@Override
	public String getResult(CallableStatement cs, int index) throws SQLException {
		log.info("使用我的TypeHandler, CallableStatement下标获取字符串");
		return cs.getString(index);
	}

	@Override
	public void setParameter(PreparedStatement ps, int index, String value, JdbcType jt) throws SQLException {

		log.info("使用我的TypeHandler");
		ps.setString(index, value);
	}

}
```    

- 在config.xml文件中注册自定义typeHandler   
```Java
<typeHandlers>
		<typeHandler jdbcType="VARCHAR" javaType="string" handler="com.learn.chapter3.typeHandler.MyStringTypeHandler"/>
</typeHandlers>
```  

- 修改映射文件   

```Java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter3.mapper.RoleMapper">
	<resultMap type="role" id="roleMap">
		<id column="id" property="id" javaType="long" jdbcType="BIGINT"/>
		<result column="role_name" property="roleName" javaType="string" jdbcType="VARCHAR"/>
		<result column="note" property="note" typeHandler="com.learn.chapter3.typeHandler.MyStringTypeHandler"/>
	</resultMap>
	
	<select id="getRole" parameterType="long" resultMap="roleMap">
		select id, role_name, note from t_role where id 	= #{id}
	</select>
	
	<select id="findRole" parameterType="string" resultMap="roleMap">
		select id, role_name, note from t_role where role_name like 
		concat('%', #{roleName javaType=string, jdbcType=VARCHAR, 
		typeHandler=com.learn.chapter3.typeHandler.MyStringTypeHandler}, '%')
	</select>
	
	<insert id="insertRole" parameterType="role">
		insert into t_role(role_name, note) values (#{roleName}, #{note})
	</insert>
	
	<delete id="deleteRole" parameterType="long">
		delete from t_role where id = #{id}
	</delete>
	
</mapper>
```  

#### 3. ObjectFactory  

当MyBatis在构建一个结果返回时，都会使用ObjectFactory（对象工厂）去构建POJO，在MyBatis中可以定制自己的对象工厂。  
一般来说使用默认的ObjectFactory即可，默认实现是DefaultObjectFactory。一般继承该类覆写方法即可。  

#### 4. environments配置环境  

```Java
<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC"/>
            <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis" />
            <property name="username" value="root"/>
            <property name="password" value="vincent"/>
            </dataSource>
		</environment>
</environments>
```  
- environments中default参数表示缺省下使用哪个数据库源配置；  
- environment元素是配置一个数据源的开始，id是标识；  
- transactionManager配置的事数据库事务，type有三种，JDBC, MANAGED（采用容器方式管理，JNDI中常用），自定义方式；  
- dataSource是数据库连接方式，有UNPOOLED, POOLED, JNDI和自定义方式。  


#### 5. 映射器  
有四种引入方式：  
- 文件路径引入  
  
``` Java
<mappers> 
	<mapper resource="com/xx/yy/zz/roleMapper.xml"/>
</mappers>
```  

- 用包名引入  

```Java
<mappers>
	<package name="com.xx.yy.zz.po">
</mappers>
```  

- 用类注册引入  

```Java  
<mappers>
	<mapper class="com.xx.yy.zz.RoleMapper">
</mappers>
```

- 用userMapper.xml引入  

``` Java
<mappers>
	<mapper url="file:///var/mappers/com/xx/yy/zz/mapper/roleMapper.xml">
</mappers>
```  

