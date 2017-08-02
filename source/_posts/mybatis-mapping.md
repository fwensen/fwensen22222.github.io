---
title: MyBatis映射器
date: 2017-08-02 23:00:54
tags: MyBatis
categories: Database
---

### 映射器  
MyBatis的映射器提供强大的映射功能，这也是MyBatis的核心功能。  
### 1. select元素  
select元素中最主要的几个元素四parameterType、resultType、resultMap等。其中parameterType表示参数类型，可给出类别名或类全名以及MyBatis内部定义类型，resultType表示返回值类型，resultMap帮助我们执行强大的自定义映射功能。  

#### 1. 自动映射  
有这样一个参数autoMappingBehavior，当它不设置为NONE的时候，MyBatis会提供自动映射的功能，只要返回的SQL列名和JavaBean的属性一致，MyBatis就会帮助我们回填这些字段而无需任何配置。  
举例：

```Java
package com.learn.chapter4.po;

public class Role {

	private long id;
	
	private String roleName;
	
	private String note;

	public long getId() {
		return id;
	}

	public void setId(long id) {
		this.id = id;
	}

	public String getRoleName() {
		return roleName;
	}

	public void setRoleName(String roleName) {
		this.roleName = roleName;
	}

	public String getNote() {
		return note;
	}

	public void setNote(String note) {
		this.note = note;
	}
	
	public String toString() {
		return "[id: " + id + ", roleName: " + roleName + ", note: " + note + "]";
	}
}
```

roleMapper.xml文件如下：

```Java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.RoleMapper">
	<select id="getRole" parameterType="long" 
		resultType="com.learn.chapter4.po.Role">
		select id, role_name as roleName, note from t_role
		where id = #{id}	
	</select>
</mapper>
```

注意这里select语句中，role_name as roleName是必要的，若不将role_name设置别名为roleName，和Role中roleName对应，MyBatis自动映射就不能完全映射，得到的值会变成null。
这样，上面的语句便完成了自动映射，自动映射的参数可在settings元素中配置autoMappingBehavior：  
- NONE：取消自动映射；  
- PARTIAL：只会自动映射，没有嵌套结果集映射的结果集；  
- FULL：会自动映射任意复杂的结果集，但在性能上会有所下降。  
默认是PARTIAL映射。  
**如果数据库命名规范，即每个单词都用下划线分开，POJO采用驼峰式命名方法，那么也可设置mapUnderscoreToCamelCase为true完成自动映射**。

#### 2. 传递多个参数

对于传递多个参数的语句，有三种方法。

##### 1. map传递参数

```Java
<select id="findRoleMap" parameterType="map" resultMap="roleMap">
	select id, role_name, note from t_role where role_name like concat('%', #{roleName}, '%')
	and note like concat('%', #{note}, '%')
</select>
```

然后提供对于RoleDao接口，提供一个方法即可：

```Java
public List<Role> findRoleByMap(Map<String, String> params);
```

##### 2. 使用注解方式

使用注解@Param可声明参数。

```Java
public List<Role> findRoleByAnnotation(@Param("roleName") String roleName, @Param("note") String note);
```

XML文件也无需定义参数类型。

```Java
<select id="findRoleByAnnotation" resultMap="roleMap">
	select id, role_name, note from t_role where role_name like concat('%', #{roleName}, '%')
	and note like concat('%', #{note}, '%')
</select>
```

这样，当我们将参数传递到后台时，通过@Param注解提供的名称，MyBatis就知道#{roleName}代表roleName参数，但这样有个问题是当参数数量很多时，参数会变得很复杂。

##### 3. 使用JavaBean传递参数

在参数过多的情况下，MyBatis允许组织一个JavBean，通过简单的setter/getter方法设置参数。提高可读性。

```Java
package xx.yy.zz.role;
public class RoleParam {

	private String roleName;
	private String note;
	
	//setter/getter
}
```

使用JavaBean作为参数的XML文件如下：

```
<select id="findRoleByParam" parameterType="xx.yy.zz.role.RoleParam" resultMap="roleMap">
	select id, role_name, note from t_role where role_name like concat('%', #{roleName}, '%') and note like concat('%', #{note}, '%')
</select>
```

这样便可完成多参数的传递。

#### 3. 使用resultMap映射结果集

在处理简单结果映射时可使用自动映射方式，但在处理较为复杂的映射时，则需要自定义resultMap。

```Java
<resultMap type="role" id="roleResultMap">
		<id column="id" property="id" javaType="long" jdbcType="BIGINT"/>
		<result column="role_name" property="roleName" javaType="string" jdbcType="VARCHAR"/>
		<result column="note" property="note" typeHandler="com.learn.chapter3.typeHandler.MyStringTypeHandler"/>
</resultMap>
...
<select parmeterType="long" id="getRole" resultMap="roleResultMap">
	select id, role_name, note from t_role where id=#{id}
</select>
```

上面的resultMap中，定义了映射的规则，其中<id>指定了主键，<result>中，column指定了数据库列名，property指定了POJO相对应的列名。
这样select语句不再需要自动映射便可完成规则的映射。


### 2. insert元素

insert相对select元素简单得多，MyBatis在执行插入后会返回整数表示插入的记录数。

#### 1. 主键回填和自定义

可以在insert的keyProperty属性指定主键字段，同时使用useGeneratedKeys属性告诉MyBatis这个主键是否使用数据库内置策略生成。
例如在主键id列为自增字段时，可有如下设置：

```Java
<insert id="insertRole" parameterType="role" useGeneratedKeys="true" keyProperty="id">
	insert into t_role(role_name, note) values(#{roleName}, #{note})
</insert>
```

这样传入的role对象就无需设置id的值，MyBatis会用数据库的设置进行处理。


### 3. update元素和delete元素

和insert一样，也会返回一个整数表示执行后影响的记录条数。

### 4. 特殊字符串替换和处理(#和$)

- 在参数#{name}中，MyBatis会用创建预编译的语句，然后MyBatis设值；
- 而在参数${name}中，可传递SQL语句本身，而不是SQL所需的参数，这样MyBatis帮我们转译name，变成直出，而不是作为参数设置。
