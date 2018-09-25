---
layout: post
title:  "mybatis"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

### 一.mybatis 基本配置
最近几天一直在学习mybatis，看了一些源码，本文讲述mybatis的一些基本配置和基本的用法和注意到一些细节。个人时间和精力有限，本文属于流水账类型，不成体系，算是自己的个人笔记吧。

<!--more-->

1.本案例所使用的数据库为mysql，数据库的脚本代码如下：

```
CREATE TABLE `message` (
  `ID` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `COMMAND` varchar(16) DEFAULT NULL COMMENT '指令名称',
  `DESCRIPTION` varchar(32) DEFAULT NULL COMMENT '描述',
  `CONTENT` varchar(2048) DEFAULT NULL COMMENT '内容',
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8;


INSERT INTO `message` VALUES (1, '查看', '精彩内容', '精彩内容');
INSERT INTO `message` VALUES (2, '段子', '精彩段子', '如果你的月薪是3000块钱，请记得分成五份，一份用来买书，一份给家人，一份给女朋友买化妆品和衣服，一份请朋友们吃饭，一份作为同事的各种婚丧嫁娶的份子钱。剩下的2999块钱藏起来，不要告诉任何人');
INSERT INTO `message` VALUES (3, '新闻', '今日头条', '7月17日，马来西亚一架载有298人的777客机在乌克兰靠近俄罗斯边界坠毁。另据国际文传电讯社消息，坠毁机型为一架波音777客机，机载约280名乘客和15个机组人员。\r\n乌克兰空管部门随后证实马航MH17航班坠毁。乌克兰内政部幕僚表示，这一航班在顿涅茨克地区上空被击落。马来西亚航空公司确认，该公司从阿姆斯特丹飞往吉隆坡的MH17航班失联，并称最后与该客机取得联系的地点在乌克兰上空。图为马航客机坠毁现场。');
INSERT INTO `message` VALUES (4, '娱乐', '娱乐新闻', '昨日，邓超在微博分享了自己和孙俪的书法。夫妻同样写幸福，但差距很大。邓超自己都忍不住感慨字丑：左边媳妇写的。右边是我写的。看完我再也不幸福了。');
INSERT INTO `message` VALUES (5, '电影', '近日上映大片', '《忍者神龟》[2]真人电影由美国派拉蒙影业发行，《洛杉矶之战》导演乔纳森·里贝斯曼执导。 \r\n片中四只神龟和老鼠老师都基于漫画和卡通重新绘制，由动作捕捉技术实现。\r\n其中皮特·普劳泽克饰演达芬奇(武器：武士刀)，诺尔·费舍饰演米开朗基罗(武器：双节棍)，阿伦·瑞奇森饰演拉斐尔(武器：铁叉)，杰瑞米·霍华德饰演多拉泰罗(武器：武士棍)。\r\n该片计划于2014年8月8日在北美上映。');
INSERT INTO `message` VALUES (6, '彩票', '中奖号码', '查啥呀查，你不会中奖的！');
```
2.创建实体类

```
public class Message {
	/**
	 * 主键
	 */
	private String id;
	/**
	 * 指令名称
	 */
	private String command;
	/**
	 * 描述
	 */
	private String description;
	/**
	 * 内容
	 */
	private String content;
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getCommand() {
		return command;
	}
	public void setCommand(String command) {
		this.command = command;
	}
	public String getDescription() {
		return description;
	}
	public void setDescription(String description) {
		this.description = description;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
}


```

 3.去[官网](http://blog.mybatis.org/)下载mybatis ，导入相应的jar包。
 
 
 创建配置文件configuration.xml
 
 ```
 
 <?xml version="1.0" encoding="UTF-8" ?>
 <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
<!-- 
  <settings>
    <setting name="useGeneratedKeys" value="false"/>
    <setting name="useColumnLabel" value="true"/>
  </settings>

  <typeAliases>
    <typeAlias alias="UserAlias" type="org.apache.ibatis.submitted.complex_property.User"/>
  </typeAliases> -->
  
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC">
        <property name="" value=""/>
      </transactionManager>
      <dataSource type="UNPOOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/micro_message"/>
        <property name="username" value="root"/>
        <property name="password" value="123"/>
      </dataSource>
    </environment>
  </environments>
  
  <mappers>
    <mapper resource="com/imooc/config/sqlxml/Message.xml"/>
    <mapper resource="com/imooc/config/sqlxml/Command.xml"/>
    <mapper resource="com/imooc/config/sqlxml/CommandContent.xml"/>
  </mappers>

</configuration>
 ```
 
 这个配置文件，需要一个对象跟数据库交互,这个对象是sqlsession,下面讲述sqlsession.
 

### 二.Sqlsession对象
Sqlsession的作用：

* 1. 向sql 语句传入参数
* 2. 执行 sql语句
* 3. 获取执行sql语句的结果
* 4. 事物的控制

如何获取Sqlsession：

* 1. 通过获取配置文件获取数据库连接相关信息
* 2. 通过配置信息构建 sqlSessionFactory 的对象
* 3. 通过sqlsessionFactory大家数据库会话。

```
public SqlSession getSqlSession() throws IOException {
		// 通过配置文件获取数据库连接信息
		Reader reader = Resources.getResourceAsReader("com/forzp/config/Configuration.xml");
		// 通过配置信息构建一个SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
		// 通过sqlSessionFactory打开一个数据库会话
		SqlSession sqlSession = sqlSessionFactory.openSession();
		return sqlSession;
	}

```

3.SQL基本配置、执行



Message.xml文件配置：

```
<mapper namespace="Message">

  <resultMap type="com.imooc.bean.Message" id="MessageResult">
    <id column="ID" jdbcType="INTEGER" property="id"/>
    <result column="COMMAND" jdbcType="VARCHAR" property="command"/>
    <result column="DESCRIPTION" jdbcType="VARCHAR" property="description"/>
    <result column="CONTENT" jdbcType="VARCHAR" property="content"/>
  </resultMap>

</mapper>

<select id="queryMessageList" parameterType="com.imooc.bean.Message" resultMap="MessageResult">
    select <include refid="columns"/> from MESSAGE
    <where>
    	<if test="command != null and !"".equals(command.trim())">
	    	and COMMAND=#{command}
	    </if>
	    <if test="description != null and !"".equals(description.trim())">
	    	and DESCRIPTION like '%' #{description} '%'
	    </if>
    </where>
  </select>

```

其中使用到了ONGL表达式:


![WX20161225-220301@2x.png](http://upload-images.jianshu.io/upload_images/2279594-e0f33461de4654fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![WX20161225-220208@2x.png](http://upload-images.jianshu.io/upload_images/2279594-469157fcf09f72e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




在configuration.xml中配置

```
 <mappers>
    <mapper resource="com/imooc/config/sqlxml/Message.xml"/>
  
  </mappers>


```

执行：

```
/**
	 * 根据查询条件查询消息列表
	 */
	public List<Message> queryMessageList(String command,String description) {
		DBAccess dbAccess = new DBAccess();
		List<Message> messageList = new ArrayList<Message>();
		SqlSession sqlSession = null;
		try {
			sqlSession = dbAccess.getSqlSession();
			Message message = new Message();
			message.setCommand(command);
			message.setDescription(description);
			// 通过sqlSession执行SQL语句
			messageList = sqlSession.selectList("Message.queryMessageList",message);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			if(sqlSession != null) {
				sqlSession.close();
			}
		}
		return messageList;
	}

```

### 三、log4j动态调试SQL
由于mybatis的sql语句写在了xml中，导致调试比较困难，不能打断点。这时需要日志去调。

下载log4j jar包,并配置log4j.propertites

```
log4j.rootLogger=DEBUG,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
log4j.logger.org.apache=INFO

```

log的级别：

>log.debug
>
>log.info

>log.warm

>log.error
>
级别从小到大，也就是og4j.rootLogger=DEBUG；可以输出所有的日志类型；og4j.rootLogger=info ，只能够输出　info,warm ,error

要想mybtis  显示日志，必须log4j.rootLogger=DEBUG；
项目中配置了log4j，mybatis 会自动应用log4j

### 四、一些其他的知识点
 mybatis sql配置文件常用的标签如下：
 
![WX20161226-213750@2x.png](http://upload-images.jianshu.io/upload_images/2279594-96c5b9f6f1dc1bb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


mapper文件下的标签含义：

nameSpace  标签区分可以用来区分sql文件

parameterType	将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。

resultMap	外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，对其有一个很好的理解的话，许多复杂映射的情形都能迎刃而解。使用 resultMap 或 resultType，但不能同时使用。


### 五、一对多sql xml的配置

java实体内Command包含了一个List<CommandContent> contentList集合。

```

public class Command {
	/**
	 * 主键
	 */
	private String id;
	/**
	 * 指令名称
	 */
	private String name;
	/**
	 * 描述
	 */
	private String description;
	/**
	 * 一条指令对应的自动回复内容列表
	 */
	private List<CommandContent> contentList;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
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
	public List<CommandContent> getContentList() {
		return contentList;
	}
	public void setContentList(List<CommandContent> contentList) {
		this.contentList = contentList;
	}
}

```

在mapper中配置：

```
<mapper namespace="Command">
  <resultMap type="com.imooc.bean.Command" id="Command">
    <id column="C_ID" jdbcType="INTEGER" property="id"/>
    <result column="NAME" jdbcType="VARCHAR" property="name"/>
    <result column="DESCRIPTION" jdbcType="VARCHAR" property="description"/>
    <collection property="contentList"  resultMap="CommandContent.Content"/>
  </resultMap>
  
  <select id="queryCommandList" parameterType="com.imooc.bean.Command" resultMap="Command">
    select a.ID C_ID,a.NAME,a.DESCRIPTION,b.ID,b.CONTENT,b.COMMAND_ID
    from COMMAND a left join COMMAND_CONTENT b
    on a.ID=b.COMMAND_ID
    <where>
    	<if test="name != null and !"".equals(name.trim())">
	    	and a.NAME=#{name}
	    </if>
	    <if test="description != null and !"".equals(description.trim())">
	    	and a.DESCRIPTION like '%' #{description} '%'
	    </if>
    </where>
  </select>
  
</mapper>

```

CommandContent.xml配置

```
<mapper namespace="CommandContent">
  <resultMap type="com.imooc.bean.CommandContent" id="Content">
    <id column="ID" jdbcType="INTEGER" property="id"/>
    <result column="CONTENT" jdbcType="VARCHAR" property="content"/>
    <result column="COMMAND_ID" jdbcType="VARCHAR" property="commandId"/>
  </resultMap>
</mapper>

```

经过这样的配置，在DAO层执行，就可以取出command中的属性。

六.取自增长 key


```
<insert id="insert" parameterType="com.imooc.bean.Command" useGeneratedKeys="true" keyProperty="id">
  insert into Command (name,description) values(#{name},#{description});
 </insert>

```

七、一点细节

```
#{} 和${}区别：
#{}相当于preparedStatment;
${}相当于statment,它主要用于排序oderby 

```
另外：

敲黑板划重点，mybatis的所有知识都可以在官网上都有，建议看十遍，……^_^，地址：[http://www.mybatis.org/mybatis-3/zh/configuration.html](http://www.mybatis.org/mybatis-3/zh/configuration.html)