---
title: MyBatis resultMap
date: 2017-08-03 23:20:14
tags: MyBatis
categories: Database
---

resultMap是MyBatis里面最复杂的元素，它的作用是定义映射规则，级联的更新、定制类型转化器等。

### 1. resultMap元素构成

resultMap元素里面有以下元素构成：

```Java
<resultMap>
	<!--定义构造器-->
	<constructor>
		<idArg>
		</arg>
	</constructor>
	<id/>
	<result/>
	<!--一对一级联-->
	<association/>
	<!--一对多级联-->
	<collection/>
	<!--选择器-->
	<discriminator>
		<case/>
	</discrimitor>
```

其中constructor元素用于配置构造方法。一个POJO可能不存在没有参数的构造器，这时我们就可用constructor进行配置。

```Java
<resultMap ...>
	<constructor>
		<idArg column="id" javaType="int"/>
		<arg column="role_name" javaType="string">
	</constructor>
...
</resultMap>
```

</id>表示哪个列是主键，允许多个主键（联合主键），result是配置POJO到SQL列名的映射关系。其中reuslt和id都有以下属性：property、column、javaType、jdbcType、typeHandler等，此外还有association、collection和discrimator元素等。

### 2. 使用POJO存储结果集

任何select语句都可用map存储

```Java
<select id="findColorByNote" parameterType="string" resultType="map">
	select id, color, note from t_color where note like concat('%', #{note}, '%')
</select>
```

但这种方式意味着可读性的下降，所以更推荐使用POJO方式。

使用POJO的方式见前一篇博客。

### 3. 级联

在级联中存在三种对应关系，一对一、一对多和多对多关系。

#### 1. assocation一对一级联

以学生信息级联为例，学校里学生(Student)和学生证(Selfcard)是一对一关系，所以可以建立一个StudentBean和StudengSelfcardBean的POJO对象。

```Java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.StudengSelfcardMapper">
	
	<resultMap type="com.learn.chapter4.po.StudentSelfcardBean" id="studentSelfcardMap">
		<id property="id" column="id"/>
		<result property="studentId" column="student_id"/>
		<result property="native_" column="native"/>
		<result property="issueDate" column="issue_date"/>
		<result property="endDate" column="end_date"/>
		<result property="note" column="note"/>
	</resultMap>
	
	<select id="findStudentSelfcardByStudentId" parameterType="int"
		resultMap="studentSelfcardMap">
		select id, student_id, native, issue_date, end_date, note
		from t_student_selfcard where student_id = #{studentId}
	</select>
</mapper>
```

上面是studentSelfcard.xml文件，t_student_selfcard与t_student表使用student_id字段关联，findStudentSelfcardByStudentId函数通过studentId查询得到学生的学生证信息。


```Java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.StudentMapper">
	
	<resultMap type="com.learn.chapter4.po.StudentBean" id="studentMap">
		<id property="id" column="id"/>
		<result property="cnname" column="cnname"/>
		<result property="sex" column="sex" jdbcType="INTEGER" 
			javaType="com.learn.chapter4.enums.SexEnum"
			typeHandler="com.learn.chapter4.typehandler.SexTypeHandler"/>
		<result property="note" column="note"/>
		<association property="studentSelfcard" column="id" 
		select="com.learn.chapter4.mapper.StudentSelfcardMapper.findStudentcardByStudentId"/>
	</resultMap>
	
	<select id="getStudent" parameterType="int"
		resultMap="studentMap">
		select id, cnname, sex, note from t_student where id=#{id}
	</select>
</mapper>
```

上面是student.xml文件，上面着重注意的地方是association元素部分，表示通过t_student表的id字段与学生证表级联，从而select语句部分就可以通过这种级联关系，在调用getStudentt同时也可得到学生证信息。

#### 2. collection一对多级联

一个学生可能有多门课程，在学生确定的情况下，每门课程都有自己的分数，所以一个学生和课程的级联是一对多的关系。

```Java
public class LectureBean {

	private Integer id;
	private String lectureName;
	
	private String note;
	//---setter/getter
}

public class StudentLectureBean {
	private int id;
	private int studentId;
	private LectureBean lecture;
	private BigDecimal grade;
	private String note;
	// setter/getter
}
```

StudentLectureBean包含一个lecture属性用来读取的课程信息，这里可用association做一对一级联。在StudentBean里面增加一个类型为List<StudentLectureBean>的属性studentLectureList，用来保存学生课程成绩信息。这时我们用collection一对多级联。

```Java 
<!------ student.xml ----->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.StudentMapper">
	
	<resultMap type="com.learn.chapter4.po.StudentBean" id="studentMap">
		<id property="id" column="id"/>
		<result property="cnname" column="cnname"/>
		<result property="sex" column="sex" jdbcType="INTEGER" 
			javaType="com.learn.chapter4.enums.SexEnum"
			typeHandler="com.learn.chapter4.typehandler.SexTypeHandler"/>
		<result property="note" column="note"/>
		<association property="studentSelfcard" column="id" 
		select="com.learn.chapter4.mapper.StudentSelfcardMapper.findStudentcardByStudentId"/>
		<collection property="studentLectureList" column="id" 
			select="com.learn.chapter4.mapper.StudentLectureMapper.findStudentLectureByStuId" />
	</resultMap>
	
	<select id="getStudent" parameterType="int" resultMap="studentMap">
		select id, cnname, sex, note from t_student where id=#{id}
	</select>
</mapper>


<!------ studentLectureMapper.xml ----->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.StudentLectureMapper">
	
	<resultMap type="com.learn.chapter4.po.StudentLectureBean" 
		id="studentLectureMap">
		<id property="id" column="id"/>
		<result property="studentId" column="student_id"/>
		<result property="grade" column="grade"/>
		<result property="note" column="note"/>
		<association property="lecture" column="lecture_id"
		select="com.learn.chapter4.mapper.LectureMapper.getLecture"/>
	</resultMap>
	
	<select id="findStudentLectureByStuId" parameterType="int" 
	resultMap="studentLectureMap">
		select id, student_id, lecture_id, grade, note from t_student_lecture
		where student_id = #{id} 
	</select>
	
</mapper>


<!----- lecture.xml----->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.LectureMapper">
	<select id="getLecture" parameterType="int" resultMap="com.learn.chapter4.po.LectureBean">
		select id, lecture_name as lectureName, note from t_lecture where id=#{id}
	</select>
</mapper>

```

student.xml文件中使用collection关联StudentLectureBean，其中column对应SQL列名，这里用id，属性是Student的studentLectureList, 而配置的select为findStudentLectureByStuId，那么MyBatis就会启用这条语句来加载数据。这里用StudentLectureBean去级联LectureBean信息，它使用了列lecture_id作为参数，用对应的select语句进行加载。

#### 3. discriminator鉴别器级联

理解鉴别器最好的例子是switch语句，它有类似的功能。
例如我们根据sex字段得到男女学生，那么：

```Java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.learn.chapter4.mapper.StudentMapper">
	
	<resultMap type="com.learn.chapter4.po.StudentBean" id="studentMap">
		<id property="id" column="id"/>
		<result property="cnname" column="cnname"/>
		<result property="sex" column="sex" jdbcType="INTEGER" 
			javaType="com.learn.chapter4.enums.SexEnum"
			typeHandler="com.learn.chapter4.typehandler.SexTypeHandler"/>
		<result property="note" column="note"/>
		<association property="studentSelfcard" column="id" 
		select="com.learn.chapter4.mapper.StudentSelfcardMapper.findStudentcardByStudentId"/>
		<collection property="studentLectureList" column="id" 
			select="com.learn.chapter4.mapper.StudentLectureMapper.findStudentLectureByStuId" />
	
		<discriminator javaType="int" column="sex">
			<case value="1" resultMap="maleStudentMap" />
			<case value="2" resultMap="femaleStudentMap" />
		</discriminator>
	</resultMap>
	
	<resultMap type="com.learn.chapter4.po.MaleStudentBean" id="maleStudentMap" 
		extends="studentMap">
			<collection property="studentHealthMaleList" column="id"
			select="com.learn.chapter4.mapper.StudentHealthMaleMapper.findStudentHealthMaleByStuId"></collection>
	</resultMap>
	<resultMap type="com.learn.chapter4.po.FemaleStudentBean" id="femaleStudentMap" 
		extends="studentMap">
			<collection property="studentHealthFemaleList" column="id"
			select="com.learn.chapter4.mapper.StudentHealthFemaleMapper.findStudentHealthFemaleByStuId"></collection>
	</resultMap>
	<select id="getStudent" parameterType="int" resultMap="studentMap">
		select id, cnname, sex, note from t_student where id=#{id}
	</select>
</mapper>
```

这里是增加discriminator的student.xml文件，出现两种选择，根据sex值分别选择maleStudentMap和femaleStudentMap，这两个resultMap都扩展了原有的studentMap，resultMap可以继承，然后加入自己的属性。

### 4. 延迟加载

级联有N+1问题，N+1问题表示当取出一个对象时，有级联则会同时取出级联对象，例如上面取出Student会同时取出StudentSelfcard对象等等。
为解决这种问题，所以有延迟加载的解决方案。MyBatis配置中有两个全局参数lazyLoadingEnabled和aggressiveLazyLoading。

1. lazyLoadingEnabled: 延迟加载配置: <setting name="lazyLoadingEnabled" value="true">启用  

2. aggressiceLazyLoading: 对任意延迟属性的调用会使带有延迟加载属性的对象完整加载。  <setting name="aggrassiveLazyLoading" value="false">，这样就可以按需加载，而不是使用层级加载。层级加载表示加载时若是同一层的，则在加载其中一项时，也会将另外同一层的项加载出来。如何理解层级关系呢，例如与学生关联的层级有课程成绩和学生证，这里课程成绩和学生证是同一层级，若没有配置aggressiveLazyLoading元素，则在访问课程成绩时，也会去加载学生证信息。

3. 局部项单独设置的方式：可在association和collection的属性值fetchType中设置：eager或lazy。 

lazyLoadingEnabled和aggressiceLazyLoading都是setting的属性元素，可按以下方式配置：

```Java
<settings>
	<setting name="lazyLoadingEnabled" value="true">
</settings>
```
