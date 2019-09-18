---
layout:     post
title:      MybatisPlus之typeHandler使用
subtitle:   减少对业务开发中数据类型转换的关注
date:       2019-09-06
author:     WJ
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - MybatisPlus
    - 中间件
    - Java
    - 开发技巧
---

>typeHandler是Mplus中的一个注解
主要应用为参数的类型转换

这里用到的是postgreSQL，字段类型为jsonb 使用typeHandler的目的是不用自己手动转换参数；

先来看一下代码的书写格式：

###typeHandler
``Bean中使用定义``
```
 /**
        * 用户信息
        */
       @TableField(value = "user_data", typeHandler = MapJsonTypeHandler.class)
       private HashMap<String, String> userData;
```

接下来是MapJsonTypeHadnler.class的功能实现，该类继承于自定义转换类：

```
public class MapJsonTypeHandler extends JsonTypeHandler<Map> {
}


//JsonTypeHandler.class
import com.alibaba.fastjson.JSON;
import org.apache.commons.lang3.StringUtils;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;
import org.postgresql.util.PGobject;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
public class JsonTypeHandler<T> implements TypeHandler<T> {

    private Type type;
    
    public JsonTypeHandler() {
        type = ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0];
    }
    @Override
    public void setParameter(PreparedStatement ps, int i, T object, JdbcType jdbcType) throws SQLException {
        PGobject pGobject = new PGobject();
        pGobject.setType("json");
        pGobject.setValue(JSON.toJSONString(object));
        ps.setObject(i, pGobject);
    }
    @Override
    public T getResult(ResultSet rs, String columnName) throws SQLException {
        return fromJson(rs.getString(columnName));
    }
    @Override
    public T getResult(ResultSet rs, int columnIndex) throws SQLException {
        return fromJson(rs.getString(columnIndex));
    }
    @Override
    public T getResult(CallableStatement cs, int columnIndex) throws SQLException {
        return fromJson(cs.getString(columnIndex));
    }
    @SuppressWarnings("unchecked")
    private T fromJson(String json) {
        if (StringUtils.isBlank(json)) {
            return null;
        }
        try {
            return (T) JSON.parseObject(json, type);
        } catch (Exception e) {
            throw new RuntimeException("JsonTypeHandler read json error! json=" + json + ",type=" + type, e);
        }
    }
}
```

XML配置还需要添加上引用的typeHandler类
```
        <result column="user_data" property="userData"  typeHandler="com.lingban.cti.billserver.dao.handler.MapJsonTypeHandler"/>

```

那么现在基本的配置都已经完成了，那么其它类型的操作怎么写？在哪里使用呢？
###映射器
下面我要来插入一个新的东西叫做 org.mapstruct.Mapper；注解：这个叫做映射器，需要指定映射的模型；

作为入参和实体``Bean``之间的转换；
定义一个转换接口然后通过注解来指定转换器模型--
大致格式如下：
```
@Mapper(componentModel = "spring", uses = {DateConvert.class, CEnumConvert.class})
public interface CdrConvert {

    Bean req2Bean(Request req);
    Respones bean2Resp(Bean bean);

```

下面来看一下映射器模型的内容：
```
@Mapper(componentModel = "spring")
public interface DateConvert {

    default Date long2Date(long timestamp) {
        return new Date(timestamp);
    }
    default long date2Long(Date date) {
        if (date == null) {
            return 0;
        }
        return date.getTime();
    }
}
```

如果涉及到其他的类型转换，例如：Enum转Integer，date转long，等等需要操作的数据转换可以通过这种模型的方式配置进去
然后再编写业务逻辑的时候就完全不需要考虑类型转换的问题了；

那么这个东西就先说到这，可能会持续更新。。。如果谁在使用mybatisPlus或者有什么其他新鲜的东西。
欢迎留言，欢迎互相学习交流。

希望每个人的生活中都有一道属于他自己的光😸；



