---
title: MyBatis基础
date: 2017-07-31 23:38:52
tags: MyBatis
categories: Database
---

### MyBatis基础  
MyBatis由以下核心组件构成：  
1. **SqlSessionBuilder**: 它根据配置信息(XML)或代码生成SqlSessionFactory工厂接口，其中XML方式是推荐的方式，因为这种方式可避免硬编码以及方便日后维护修改，避免重复编译代码；它的作用主要是**一个构建器，一旦构建了相应的SqlSessionFactry，作用便已完结，这时Java垃圾回收器便可回收它**。    
2. **SqlSessionFactory**：生成SqlSession的工厂接口；生命期在整个MyBatis应用的生命期中，职责是创建SqlSession，为避免数据库连接的浪费，尽量使用单例模式。  
3. **SqlSession**；这是用于执行SQL的核心组件，可用于发送SQL语句并返回结果，也可获取Mapper接口；相当于JDBC的一个Connection对象，所以生命期在一个SQL事务中，它是一个线程不安全的对象，所以在涉及多线程时需注意线程安全性，同时尽量放在finally语句中以便释放。  
4. **SQL Mapper**: 由一个Java接口和XML文件（或注解）构成，给出对应SQL语句的映射规则。它的生命期在一个SqlSession事务内，是一个方法级别的东西，使用完后尽量废弃掉。   

#### SqlSessionFactory  
该对象关系到JDBC连接问题，所以一般为单例模式。使用XML实现方式如下：  
```  
<!--config.xml文件-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<typeAliases>
		<!-- 定义别名，上下文可使用role表示Role对象 -->
		<typeAlias type="xxx.yyy.Role" alias="role"/>
	</typeAliases>
	<!-- 定义数据库信息，默认使用development数据库构建环境-- >
	<environments default="development">
		<environment id="development">
			<!--采用JDBC方式 -->
			<transactionManager type="JDBC"/>
            <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis" />
            <property name="username" value="root"/>
            <property name="password" value="password"/>
            </dataSource>
		</environment>
	</environments>
	
	<mappers>
		<mapper resource="xxx/yyy/zzz/mapper/roleMapper.xml"/>
	</mappers>

</configuration>
```
映射器配置如下，放入单例类实现中：  
```
InputStream inputStream = Resources.getResourceAsStream("config.xml");
SqlSessionFactory sqlFactory = new SqlSessionFactoryBuilder().build(inputStream);  
```
这样MyBatis的解析程序会将config.xml文件配置的信息解析到Configuration类对象里面，然后利用SqlSessionFactoryBuilder读取这个对象创建SqlSessionFactory。  

#### SqlSession  
它是一个接口类，扮演着门面的作用，真正工作的事Excutor接口。  
在构建好SqlSessionFactory后，就可用工厂类生成MyBatis的门面类SqlSession，它类似于JDBC中的Connection接口对象。  

```Java  
SqlSession sqlSession = null;
try {
	sqlSession = sqlSessionFactory.openSession();
	//... do select/insert/delete/update...
	sqlSession.commit();
} catch (Exception e) {
	sqlSession.rollback();
} finally {
	if (sqlSession != null) {
		sqlSession.close();
	}
}
```    


#### SQL Mapper  
映射器由Java接口和XML文件（或注解）构成，作用如下：  
- 定义参数类型；  
- 描述缓存；  
- 描述SQL语句；  
- 定义查询结果和POJO的映射关系。  
推荐使用XML方式。  


#### 示例：  
1. Java接口    
```  
public interface RoleMapper {
	//根据id查找相应的Role,Role是一个POJO对象（有相应的setter/getter）
	public Role getRole(Long id); 
}  
```  

2. XML映射文件   
```  
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--对应映射规则的映射接口类 -->
<mapper namespace="xxx.yyy.zzz.mapper.RoleMapper">
	<!--映射接口内实现的查询方法-->
	<!--parameterType表示参数类型，resultType表示返回类型-->
	<select id="getRole" parameterType="long" resultType="role">
		<!-- #{id}表示参数，也即getRole方法的参数 -->
		select id, role_name as roleName, note from t_role where id = #{id}
	</select>
</mapper>
```    

    XML文件的作用：  
    - 这个文件在前文的config.xml中已经配置了的，见其中的<mappers/>     
    - 定义了一个命名空间为xxx.yyy.zzz.mapper.RoleMapper的SQL Mapper，这个命名空间和我们定义的接口的全限定名一致；  
    - 用一个select元素定义了一个查询SQL，见其中的注释，注意其中的role为别名。  

3. POJO类  
```Java
public class Role {

	private long id;
	private String roleName;  
	private String note;  
	// setter/getter
}
```    

4. 使用  
``` Java
RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);  
Role role = roleMapper.getRole(1L);
```   
这样便完成了一次查询。  
