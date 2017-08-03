---
title: MyBatis动态语句
date: 2017-08-04 00:38:03
tags: MyBatis
categories: Database
---

MyBatis的动态SQL功能帮助我们免去拼装复杂SQL语句的麻烦。
它有几个主要的元素。

 元素 | 作用 
 ----|----
 if  | 判断语句
 choose(when,otherwise) | 相当于Java中的case when语句
 trim | 辅助元素，用于处理一些SQL拼装问题
 foreach|循环语句，在in语句等列举条件中常用
 
 
### if元素
相当于Java中if语句，常与test属性联合使用。

```Java
<select id="findRoles" parameterType="string" resultMap="roleResultMap">
	select role_no, role_name, note from t_role where 1=1
	<if test="rolenaName != null and roleName != ''"> 
		and role_name like concat('%', #{roleName}, '%')
	</if>
</selecr>
```

上面语句的含义是我们将参数roleName传递进入到映射器中，如果这个参数为空，我们就不去构造这个条件。这里的1=1语句主要用于拼接。

### choose、when、otherwise元素

类似于Java中的switch..case..default语句。

```Java
<select id="findRiles" parameterType="role" resultMap="roleResultMap">
	select role_no, role_name, note from t_role where 1=1
	<choose>
		<when test="roleNo != null and roleNo != ''">
			and role_no = #{roleNo}
		</when>
		<when test="roleName != null and roleName != ''">
			and role_name like concat('%', #{roleName}, '%')
		</when>
		<otherwise>
			and note is not null
		</otherwise>
	</choose>
</select>
```

上面的语句表示当roleNo不为空那么条件使用role_no = #{roleNo}，如果roleNo为空，那么判断roleName是否为空，不为空则将role_name like concat('%', #{roleName}, '%')加入where语句，前面两个条件都不满足时，则将note is not null加入where语句。

### trim、where、set元素

这几个元素主要作用帮助我们拼装，例如前面的判断语句都有一个1=1语句，使用where我们可以去掉这条语句。

```Java
<select id="findRoles" parameterTypes="string" resultMap="roleResultMap">
	select role_no, role_name, note from t_role
	<where>
	<if test="roleName != null and roleName != ''">
		and role_name like concat('%', #{roleName, '%'})
	</if>
	</where>
</select>
```

只有where里面的条件成立时才加入where这个SQL到组装的SQL语句中，否则不加入。

trim可以帮助我们去掉一些特殊的SQL语法，比如常见的and, or。

```Java
<select id="findRoles" parameterTypes="string" resultMap="roleResultMap">
	select role_no, role_name, note from t_note
	<trim prefix="where" prefixOverrides="and">
		<if test="roleName != null and roleName != ''">
			and role_name like concat('%', #{roleName}, '%')
		</if>
	</trim>
</select>
```

trim元素意味着需要去掉一些特殊的字符串，prefix表示语句前缀，prefixOverrides代表你需要去掉哪种字符串，上面的写法和where基本等效。

在Hibernate中我们常常需要更新某一对象，会发送所有的字段给持久化对象，然而当我们只想更新一个字段，若发送所有属性去更新，对网络带宽消耗较大，最佳的方式是只发送主键和待更新的字段。

在MyBatis中，常常使用set元素完成仅发送更新字段的功能。

```Java
<update id="updateRole" parameterType="role">
	update t_role
	<set>
		<if test="roleName != null and roleName != ''">
			role_name = #{roleName},
		</if>
		<if test="note != null and note != ''">
			note = #{note}
		</if>
	</set>
	where role_no = #{roleNo}
</update>
```

set元素遇到都好，它会自动把对应的逗号去掉。如果我们自己编写那将是多次的判断，使用set，MyBatis就会根据参数的规则进行动态SQL组装，这样能避免全部字段更新的麻烦。

### foreach元素

foreach元素的作用是遍历集合，它能很好的支持数组、List、Set接口的集合，提供遍历的功能。

```Java
<select id="findUserBySex" resultMap="user">
	select * from t_user where sex in
	<foreach item="sex" index="index" collection="sexLisr" open="(" separator="," close=")">
		#{sex}
	<foreach>
<select>
```

item是循环中当前的元素，index是当前元素在集合中的索引下标，collection的sexList是传递进来的参数名称，可以数组、map、set等，open和close表示以什么符号将这些集合元素包装起来，separator是各个元素的间隔符。

所以若sexList为[male, female, null]，则拼装的SQL语句为select * from t_user where sex in (male, female, null)
使用时注意集合的长度，太长会影响性能。
