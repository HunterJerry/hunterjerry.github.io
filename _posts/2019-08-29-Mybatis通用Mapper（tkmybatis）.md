---
layout:     post
title:      Mybatis中间件之tkmybatis
subtitle:   快速实现单表逻辑
date:       2019-08-29
author:     WJ
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Mybatis
    - 中间件
    - Java
---


> 快速构建业务sql 提高开发效率
> 首先需要引入tk.mybatis包

![jar包](https://github.com/HunterJerry/hunterjerry.github.io/blob/master/img/tk-mybatisPic.png "jar")

```
import tk.mybatis.mapper.common.Mapper;
import tk.mybatis.mapper.common.MySqlMapper;

public interface MyMapper<T> extends Mapper<T>, MySqlMapper<T> {
}

```

``Bean``头信息要标注出对应库的表名

```
@Data
@Table(name = "user")
public class User  implements Serializable {
}
```

##### 自己编写的MyMapper接口继承tk包下的Mapper和MysqlMapper

tkmybatis工具是一个方便开发人员 ，提高开发效率的插件：

      优点：不需要手动写sql,通过通用mapper的方式来通过参数来编辑sql逻辑；
      缺点：只接受单标操作,不接受复杂的多表关联查询的操作；

一般根据主键的操作就不多说了，
正常的insert，delete，update，select均提供默认的ById实现

说一下where后条件使用的两种实现方式：
Example和Weekend

-----

代码如下：

```

PageHelper.startPage(req.getPageNo(),req.getPageSize());

User user=new User();
//Example 实现
Example example=newExample(User.class);
Example.Criteriacriteria=example.createCriteria();
//添加like条件（自编sql）
criteria.andCondition("name like'%"+user.getName()+"%'"); 
//添加like条件（使用方法）
criteria.andLike("name",user.getName);
//同上（添加等值条件）
criteria.andEqualTo("state",user.getState());
//添加区间条件
criteria.andCondition("create_time between'"+req.getStartTime()+"'and'"+req.getEndTime()+"'");

//Weekend实现 条件同上
Weekend<User>weekend=newWeekend<>(User.class);
WeekendCriteria<Campaign,Object> keywordCriteria=weekend.weekendCriteria();
keywordCriteria.andLike(User::getName,campaign.getName());
keywordCriteria.andEqualTo(User::getState,campaign.getState());
keywordCriteria.andBetween(User::getCreateTime,req.getStartTime(),req.getEndTime());
returnnewCampaignListResp(userMapper.selectByExample(weekend));
```

通过上面的代码可以看出两种形式的区别：
两种方式效果一样，我只说一下Weekend实现方式的好处在哪里：
`Weekend` 实现的好处在于不用将真实的数据库字段暴露在外面，而是交给映射`Bean`去转换处理
这样对于数据库字段的变更我们只需要维护映射`Bean`就OK了！

##注意：
`Weekend支持java8新语法可以使用::直接引用，但只有高一些的版本支持，3.5+从哪个版本开始支持的暂不知`


