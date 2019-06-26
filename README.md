﻿# mybatis学习笔记
---
## 简介
MyBatis可以简化JDBC操作，实现数据的持久化，需要的jar包是mybatis的jar包和数据库的jar包，需要conf.xml和mapper.xml（和接口放在同一个包中）  
## 配置  
conf.xml:配置数据库信息和需要加载的映射文件，可以将数据库信息保存到另外的文件中，例如db.properties，再在配置文件中通过
```<properties resource="db.properties"/>```引入。然后通过mapper.xml将表映射成为类。mapper.xml通常和表映射成的类放在一起（dao层的接口的实现类可以改名为xxxmapper.class）,mapper文件中首先需要通过namespace引入映射类，然后定义好各种CRUD语句。此映射文件需要在配置文件中加载进去。如果配置文件中使用的事务方式为 jdbc,则需要手工commit提交，即session.commit();完整例子如下：  
```
    public static void queryStudentById() throws IOException {
        //加载mybatis配置文件（为了访问数据库）
        Reader reader = Resources.getResourceAsReader("conf.xml");
        //创建session工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //创建statement语句
        String statement = "com.entity.studentMapper.queryStudentById";
        //加载语句,返回查询结果集
        Student student = sqlSession.selectOne(statement,10001);
        //打印结果集
        System.out.println(student);
        sqlSession.close();
    }
```  
## 输入参数  

类型为简单类型（8个基本类型+String）　　

#{}，$ {}的区别 ：#{任意值}，${value} ，并且其中的标识符只能是value，用#{}可以防止sql注入的问题　　

## 输出参数  
 如果返回值类型是一个 对象（如Student），则无论返回一个、还是多个，把resultType都写成org.lanqiao.entity.Student，即 resultType="org.lanqiao.entity.Student"  

一般基于接口开发，接口中的方法必须遵循以下约定：  
		 1.方法名和mapper.xml文件中标签的id值相同  
		 2.方法的 输入参数 和mapper.xml文件中标签的 parameterType类型一致 (如果mapper.xml的标签中没有 parameterType，则说明方法没有输入参数)  
		 3.方法的返回值  和mapper.xml文件中标签的 resultType类型一致 （无论查询结果是一个 还是多个（student、List<Student>），在mapper.xml标签中的resultType中只写 一个（Student）；如果没有resultType，则说明方法的返回值为void）  
要实现 接口中的方法  和  Mapper.xml中SQL标签一一对应，还需要以下一点：namespace的值 ，就是  接口的全类名（ 接口 - mapper.xml 一一对应）  

## 匹配的过程：（约定的过程）
1.根据 接口名 找到 mapper.xml文件（根据的是namespace=接口全类名）  
2.根据 接口的方法名 找到 mapper.xml文件中的SQL标签 （方法名=SQL标签Id值）  
执行过程：```StudentMapper studentMapper = session.getMapper(StudentMapper.class) ;  studentMapper.方法();```