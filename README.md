# mybatis-sql-log

<a name="mybatis-sql-log"></a>
# mybatis-sql-log

> mybatis-sql-log 主要是为了打印mybatis 完整的sql语句，通过mybaits 提供的插件的方式进行拦截，
> 获取内部执行的sql，并将完整的sql语句打印出来。


<a name="VIcSM"></a>
## 1、使用

```xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
         <dependency>
            <groupId>com.github.WangJi92</groupId>
            <artifactId>mybatis-sql-log</artifactId>
            <version>版本</version>
        </dependency>
```
mybats.print=true 使用spring boot 工程集成
```xml
mybats.print=true
server.port = 7012
#数据库连接
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/test
spring.datasource.username = root
spring.datasource.password = mysql-root
spring.datasource.driver-class-name= com.mysql.jdbc.Driver
spring.datasource.hikari.connection-test-query = SELECT 1
```
效果

```xml
2019-08-31 12:41:35.975  INFO 3258 --- [nio-7012-exec-3] s.b.a.MybatisSqlCompletePrintInterceptor : SQL:select name, age, type from user WHERE ( name = /*__frch_criterion_1.value*/'汪吉' )    执行耗时=6
```

或者通过mybatis-config原生配置处理
```text
<!-- mybatis-config.xml -->
<plugins>
  <plugin interceptor="com.mybatis.spring.boot.autoconfigure.MybatisSqlCompletePrintInterceptor">
  </plugin>
</plugins
```

<a name="d5d3a790"></a>
## 2、mybatis 官方插件

- [插件（plugins）](http://www.mybatis.org/mybatis-3/zh/configuration.html#plugins)<br />
MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：<br />
Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)<br />
ParameterHandler (getParameterObject, setParameters)<br />
ResultSetHandler (handleResultSets, handleOutputParameters)<br />
StatementHandler (prepare, parameterize, batch, update, query)

<a name="31742b2c"></a>
## 3、相关参考 Documentation

- [Mybatis插件-sql日志美化输出](https://my.oschina.net/junjunyuanyuankeke/blog/1975439)
- [MyBatis插件及示例----打印每条SQL语句及其执行时间](https://www.cnblogs.com/Xrq730/P/6972268.Html)

上面的两篇文章整体上思路都是一致的主要是在处理参数的问题上不是很完善。这里通过mybatis 自带的ParameterHandler处理参数就比较的简单，<br />而不是通过各种复杂的判断进行处理。org/apache/ibatis/scripting/defaults/DefaultParameterHandler 这里提供了很完善的参数处理。<br />如下的代码，可以清晰的了解整个参数的解析过程，无论是动态的参数，还是对象参数，都可以完善的处理参数的解析过程。

```
public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
          String propertyName = parameterMapping.getProperty();
          if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } else {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          JdbcType jdbcType = parameterMapping.getJdbcType();
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
          } catch (TypeException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          } catch (SQLException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          }
        }
      }
    }
  }
```

<a name="9a0fb259"></a>
## 4、mybatis在执行期间，主要有四大核心接口对象：

执行器Executor，执行器负责整个SQL执行过程的总体控制。<br />参数处理器ParameterHandler，参数处理器负责PreparedStatement入参的具体设置。<br />语句处理器StatementHandler，语句处理器负责和JDBC层具体交互，包括prepare语句，执行语句，以及调用ParameterHandler.parameterize()设置参数。<br />结果集处理器ResultSetHandler，结果处理器负责将JDBC查询结果映射到java对象。

了解了这四大对象，其实我们的处理本次这个插件十分的重要，插件对象可以在Executor、或者StatementHandler两个上面处理。
