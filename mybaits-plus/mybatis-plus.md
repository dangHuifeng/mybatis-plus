[TOC]



[在线笔记](https://www.yuque.com/docs/share/0112fd84-de5f-433b-996a-b9d9cd87dd36?# 《MyBatisPlus(SpringBoot版)--2022》)



# 1、快速开始

## 1.1、配置数据库

```sql
USE mybatis_plus;
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
   id BIGINT(20) NOT NULL COMMENT '主键ID',
   name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
   age INT(11) NULL DEFAULT NULL COMMENT '年龄',
   email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
   PRIMARY KEY (id)
);
DELETE FROM user;
INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```





## 1.2、依赖



```xml
<!--MyBatis-plus启动器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>

<!--lombok用于简化实体类开发-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<!--mysql驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.3</version>
</dependency>
```





## 1.3、application.yaml



```yaml
server:
  port: 6060

spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/mybatis_plus
    type: com.alibaba.druid.pool.DruidDataSource
```



### 1.3.1、注意

```
1、驱动类driver-class-name
spring boot 2.0(内置jdbc5驱动)，驱动类使用:
driver-class-name: com.mysql.jdbc.Driver
spring boot 2.1及以上(内置jdbc8驱动)，驱动类使用:
driver-class-name: com.mysql.cj.jdbc.Driver
否则运行测试用例的时候会有 WARN 信息

2、连接地址url
MySQL5.7版本的url:
jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-8&useSSL=false
MySQL8.0版本的url:
jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=GMT%2B8&characterEncoding=utf-8&useSSL=false
否则运行测试用例报告如下错误:
java.sql.SQLException: The server time zone value 'ÖÐ1ú±ê×1⁄4Ê±1⁄4ä' is unrecognized or represents more
```



## 1.4、代码

### 1.4.1、启动类

```java
@SpringBootApplication
// 扫描mapper接口所在的包
@MapperScan("com.atguigu.mybatisplus.mapper")
public class MybatisplusApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisplusApplication.class, args);
    }

}
```



### 1.4.2、pojo



```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {

    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```



### 1.4.3、Mapper

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```



### 1.4.4、Test

```java
@SpringBootTest
class MybatisplusApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelectList() {
        // 通过条件构造器查询一个List集合，若没有条件，则可以设置null为参数
        List<User> list = userMapper.selectList(null);
        list.forEach(System.out::println);
    }
}
```



### 1.4.5、日志

```yaml
mybatis-plus:
  configuration:
    # log.info
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```





# 2、基本CURD

## 2.1、BashMapper

> ```java
> 
> package com.baomidou.mybatisplus.core.conditions;
> 
> import com.baomidou.mybatisplus.core.conditions.segments.MergeSegments;
> import com.baomidou.mybatisplus.core.conditions.segments.NormalSegmentList;
> import com.baomidou.mybatisplus.core.metadata.TableFieldInfo;
> import com.baomidou.mybatisplus.core.metadata.TableInfo;
> import com.baomidou.mybatisplus.core.metadata.TableInfoHelper;
> import com.baomidou.mybatisplus.core.toolkit.*;
> 
> import java.util.Objects;
> 
> /**
>  * 条件构造抽象类
>  *
>  * @author hubin
>  * @since 2018-05-25
>  */
> @SuppressWarnings("all")
> public abstract class Wrapper<T> implements ISqlSegment {
> 
>     /**
>      * 实体对象（子类实现）
>      *
>      * @return 泛型 T
>      */
>     public abstract T getEntity();
> 
>     public String getSqlSelect() {
>         return null;
>     }
> 
>     public String getSqlSet() {
>         return null;
>     }
> 
>     public String getSqlComment() {
>         return null;
>     }
> 
>     public String getSqlFirst() {
>         return null;
>     }
> 
>     /**
>      * 获取 MergeSegments
>      */
>     public abstract MergeSegments getExpression();
> 
>     /**
>      * 获取自定义SQL 简化自定义XML复杂情况
>      * <p>
>      * 使用方法: `select xxx from table` + ${ew.customSqlSegment}
>      * <p>
>      * 注意事项:
>      * 1. 逻辑删除需要自己拼接条件 (之前自定义也同样)
>      * 2. 不支持wrapper中附带实体的情况 (wrapper自带实体会更麻烦)
>      * 3. 用法 ${ew.customSqlSegment} (不需要where标签包裹,切记!)
>      * 4. ew是wrapper定义别名,不能使用其他的替换
>      */
>     public String getCustomSqlSegment() {
>         MergeSegments expression = getExpression();
>         if (Objects.nonNull(expression)) {
>             NormalSegmentList normal = expression.getNormal();
>             String sqlSegment = getSqlSegment();
>             if (StringUtils.isNotBlank(sqlSegment)) {
>                 if (normal.isEmpty()) {
>                     return sqlSegment;
>                 } else {
>                     return Constants.WHERE + StringPool.SPACE + sqlSegment;
>                 }
>             }
>         }
>         return StringPool.EMPTY;
>     }
> 
>     /**
>      * 查询条件为空(包含entity)
>      */
>     public boolean isEmptyOfWhere() {
>         return isEmptyOfNormal() && isEmptyOfEntity();
>     }
> 
>     /**
>      * 查询条件不为空(包含entity)
>      */
>     public boolean nonEmptyOfWhere() {
>         return !isEmptyOfWhere();
>     }
> 
>     /**
>      * 查询条件为空(不包含entity)
>      */
>     public boolean isEmptyOfNormal() {
>         return CollectionUtils.isEmpty(getExpression().getNormal());
>     }
> 
>     /**
>      * 查询条件为空(不包含entity)
>      */
>     public boolean nonEmptyOfNormal() {
>         return !isEmptyOfNormal();
>     }
> 
>     /**
>      * 深层实体判断属性
>      *
>      * @return true 不为空
>      */
>     public boolean nonEmptyOfEntity() {
>         T entity = getEntity();
>         if (entity == null) {
>             return false;
>         }
>         TableInfo tableInfo = TableInfoHelper.getTableInfo(entity.getClass());
>         if (tableInfo == null) {
>             return false;
>         }
>         if (tableInfo.getFieldList().stream().anyMatch(e -> fieldStrategyMatch(entity, e))) {
>             return true;
>         }
>         return StringUtils.isNotBlank(tableInfo.getKeyProperty()) ? Objects.nonNull(ReflectionKit.getFieldValue(entity, tableInfo.getKeyProperty())) : false;
>     }
> 
>     /**
>      * 根据实体FieldStrategy属性来决定判断逻辑
>      */
>     private boolean fieldStrategyMatch(T entity, TableFieldInfo e) {
>         switch (e.getWhereStrategy()) {
>             case NOT_NULL:
>                 return Objects.nonNull(ReflectionKit.getFieldValue(entity, e.getProperty()));
>             case IGNORED:
>                 return true;
>             case NOT_EMPTY:
>                 return StringUtils.checkValNotNull(ReflectionKit.getFieldValue(entity, e.getProperty()));
>             case NEVER:
>                 return false;
>             default:
>                 return Objects.nonNull(ReflectionKit.getFieldValue(entity, e.getProperty()));
>         }
>     }
> 
>     /**
>      * 深层实体判断属性
>      *
>      * @return true 为空
>      */
>     public boolean isEmptyOfEntity() {
>         return !nonEmptyOfEntity();
>     }
> 
>     /**
>      * 获取格式化后的执行sql
>      *
>      * @return sql
>      * @since 3.3.1
>      */
>     public String getTargetSql() {
>         return getSqlSegment().replaceAll("#\\{.+?}", "?");
>     }
> 
>     /**
>      * 条件清空
>      *
>      * @since 3.3.1
>      */
>     abstract public void clear();
> }
> ```
>



## 2.2、插入

```java
@Test
void testInsert(){
    User user = new User(null,"",20,"12345@qq.com");
    int insert = userMapper.insert(user);
    log.info("插入{}条数据，影响行数{}",insert,insert);
}
```



【mybatis-plus中添加id是使用**雪花算法**计算的，不是+1】



## 2.3、删除



> ```java
> /**
>  * 根据 ID 删除
>  *
>  * @param id 主键ID
>  */
> int deleteById(Serializable id);
> 
> /**
>  * 根据 columnMap 条件，删除记录
>  *
>  * @param columnMap 表字段 map 对象
>  */
> int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
> 
> /**
>  * 根据 entity 条件，删除记录
>  *
>  * @param queryWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
>  */
> int delete(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 删除（根据ID 批量删除）
>  *
>  * @param idList 主键ID列表(不能为 null 以及 empty)
>  */
> int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
> ```
>





```java
void testDel(){
    int i = userMapper.deleteById(6);//删除id为6的
    log.info("删除{}条数据，影响行数{}",i,i);
    //批量删除
    List<Integer> list = Arrays.asList(6, 7, 8);
    int i1 = userMapper.deleteBatchIds(list);
    //删除id=6 and name = '张三' and age = 20的数据
    HashMap<String, Object> map = new HashMap<>();
    map.put("id",6);
    map.put("name","张三");
    map.put("age",20);
    userMapper.deleteByMap(map);
}
```





## 2.4、update

```java
/**
 * 根据 ID 修改
 *
 * @param entity 实体对象
 */
int updateById(@Param(Constants.ENTITY) T entity);

/**
 * 根据 whereEntity 条件，更新记录
 *
 * @param entity        实体对象 (set 条件值,可以为 null)
 * @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
 */
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
```



```java
@Test
public void testUpdate(){
    //根据对象  id 进行修改
    // update user set name = '张三' ... where id = 6;
    User user = new User(1L, "张三", 20, "123@qq.com");
    int i = userMapper.updateById(user);
    
}
```





## 2.5、select

> ```java
> /**
>  * 根据 ID 查询
>  *
>  * @param id 主键ID
>  */
> T selectById(Serializable id);
> 
> /**
>  * 查询（根据ID 批量查询）
>  *
>  * @param idList 主键ID列表(不能为 null 以及 empty)
>  */
> List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
> 
> /**
>  * 查询（根据 columnMap 条件）
>  *
>  * @param columnMap 表字段 map 对象
>  */
> List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
> 
> /**
>  * 根据 entity 条件，查询一条记录
>  *
>  * @param queryWrapper 实体对象封装操作类（可以为 null）
>  */
> T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 根据 Wrapper 条件，查询总记录数
>  *
>  * @param queryWrapper 实体对象封装操作类（可以为 null）
>  */
> Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 根据 entity 条件，查询全部记录
>  *
>  * @param queryWrapper 实体对象封装操作类（可以为 null）
>  */
> List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 根据 Wrapper 条件，查询全部记录
>  *
>  * @param queryWrapper 实体对象封装操作类（可以为 null）
>  */
> List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 根据 Wrapper 条件，查询全部记录
>  * <p>注意： 只返回第一个字段的值</p>
>  *
>  * @param queryWrapper 实体对象封装操作类（可以为 null）
>  */
> List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 根据 entity 条件，查询全部记录（并翻页）
>  *
>  * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
>  * @param queryWrapper 实体对象封装操作类（可以为 null）
>  */
> <E extends IPage<T>> E selectPage(E page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> 
> /**
>  * 根据 Wrapper 条件，查询全部记录（并翻页）
>  *
>  * @param page         分页查询条件
>  * @param queryWrapper 实体对象封装操作类
>  */
> <E extends IPage<Map<String, Object>>> E selectMapsPage(E page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
> ```





> ```java
> @Test
> public void testSelect(){
>     //SELECT id,name,age,email FROM user WHERE id=?
>     User user = userMapper.selectById(1L);
>     System.out.println(user);
>     //SELECT id,name,age,email FROM user WHERE id IN ( ? , ? , ? )
>     List<User> users = userMapper.selectBatchIds(Arrays.asList(1L, 2L, 3L));
>     users.forEach(System.out::println);
>     //SELECT id,name,age,email FROM user WHERE name = ? AND age = ?
>     HashMap<String, Object> map = new HashMap<>();
>     map.put("name","张三");
>     map.put("age",20);
>     List<User> users1 = userMapper.selectByMap(map);
>     users1.forEach(System.out::println);
>     //SELECT id,name,age,email FROM user
>     userMapper.selectList(null).forEach(System.out::println);
> }
> ```





## 2.6、自定义功能

​	

在resource下建立mapper文件夹

在mapper下定义xml配置文件【默认】



【自定义】

```yaml
mybatis-plus:
  configuration:
    # log.info
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath:mapper/*.xml
```





```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    /**
     * 根据id查询用户信息为map集合
     * @param id
     * @return
     */
    Map<String,Object> selectMapByID(long id);
}
```



```xml
<mapper namespace="com.example.mybatisplus.mapper.UserMapper">
<!--    Map<String,Object> selectMapByID(long id);-->
    <select id="selectMapByID" resultType="map">
        select id,name,age,email from user where id = #{id}
    </select>
</mapper>
```





# 3、IService

说明:

- 通用 Service CRUD 封装[IService (opens new window)](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
- 泛型 `T` 为任意实体对象
- 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
- 对象 `Wrapper` 为 [条件构造器](https://baomidou.com/01.指南/02.核心功能/wrapper.html)

```
get   查询

save  添加

remove  删除

update  修改

...    ...

```



### 3.1、ServiceImpl

```java
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> 
```





## 3.2、自定义Service



```java
@Service
public interface UserService extends IService<User> {
}
```



```java
public class UserServiceImpl extends ServiceImpl<UserMapper, User> 
        implements UserService{
}
```



### 3.3、Service测试

> ```java
> @SpringBootTest
> @Slf4j
> public class MybatisServiceTest {
>     @Autowired
>     UserService userService;
> 
>     @Test
>     public void count(){
>         //SELECT COUNT( * ) FROM user
>         int count = userService.count();
>         log.info("总记录数{}",count);
>     }
>     @Test
>     public void saveBatch(){
>         //INSERT INTO user ( id, name, age ) VALUES ( ?, ?, ? )
>         List<User> users = new ArrayList<>();
>         users.add(new User(null,"张三",20,null));
>         users.add(new User(null,"李四",20,null));
>         boolean b = userService.saveBatch(users);
>         if (b){
>             log.info("操作成功");
>         }else {
>             log.error("操作失败");
>         }
>     }
> 
> 
> }
> ```





# 4、注解

## 4.1、@TableName



> ```
> 经过以上的测试，在使用MyBatis-Plus实现基本的CRUD时，我们并没有指定要操作的表，只是在 Mapper接口继承BaseMapper时，设置了泛型User，而操作的表为user表 
> 
> 由此得出结论，MyBatis-Plus在确定操作的表时，由BaseMapper的泛型决定，即实体类型决 定，且默认操作的表名和实体类型的类名一致
> ```



在实体类类型上添加**@TableName("t_user")**，标识实体类对应的表，即可成功执行SQL语句



> ```yaml
> mybatis-plus:
>   configuration:
>     # 配置MyBatis日志
>     log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
>   # 设置MyBatis-Plus的全局配置
>   global-config:
>     db-config:
>       # 设置实体类所对应的表的统一前缀
>       table-prefix: t_
> ```



## 4.2、@TableID

> ```
> 经过以上的测试，MyBatis-Plus在实现CRUD时，会默认将id作为主键列，并在插入数据时，默认 基于雪花算法的策略生成id
> ```

### 4.2.1、问题

> ```
> 若实体类和表中表示主键的不是id，而是其他字段，例如uid，MyBatis-Plus会自动识别uid为主 键列吗？ 
> 我们实体类中的属性id改为uid，将表中的字段id也改为uid，测试添加功能
> 
> 程序抛出异常，Field 'uid' doesn't have a default value，说明MyBatis-Plus没有将uid作为主键 赋值
> ```



### 4.2.2、@TableId注解



> ```java
> @Documented
> @Retention(RetentionPolicy.RUNTIME)
> @Target({ElementType.FIELD, ElementType.ANNOTATION_TYPE})
> public @interface TableId {
> 
>     /**
>      * 字段名（该值可无）
>      */
>     String value() default "";
> 
>     /**
>      * 主键类型
>      * {@link IdType}
>      */
>     IdType type() default IdType.NONE;
> }
> ```



```java
@TableName("user")
public class User  {
    @TableId(value = "id",type = IdType.AUTO)
    //将id对应的字段标记为主键
    //@TableId(value = "uid")
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```



@TableId 将id对应的字段标记为主键

@TableId 的 **value** 属性设置表示主键的字段名【当表的字段和类的属性不同名】

@TableId 的 **Type** 属性

> ```java
> public enum IdType {
>     /**
>      * 数据库ID自增
>      * <p>该类型请确保数据库设置了 ID自增 否则无效</p>
>      */
>     AUTO(0),
>     /**
>      * 该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
>      */
>     NONE(1),
>     /**
>      * 用户输入ID
>      * <p>该类型可以通过自己注册自动填充插件进行填充</p>
>      */
>     INPUT(2),
> 
>     /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
>     /**
>      * 分配ID (主键类型为number或string）,
>      * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(雪花算法)
>      *
>      * @since 3.3.0
>      */
>     ASSIGN_ID(3),
>     /**
>      * 分配UUID (主键类型为 string)
>      * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(UUID.replace("-",""))
>      */
>     ASSIGN_UUID(4),
>     /**
>      * @deprecated 3.3.0 please use {@link #ASSIGN_ID}
>      */
>     @Deprecated
>     ID_WORKER(3),
>     /**
>      * @deprecated 3.3.0 please use {@link #ASSIGN_ID}
>      */
>     @Deprecated
>     ID_WORKER_STR(3),
>     /**
>      * @deprecated 3.3.0 please use {@link #ASSIGN_UUID}
>      */
>     @Deprecated
>     UUID(4);
> 
>     private final int key;
> 
>     IdType(int key) {
>         this.key = key;
>     }
> }
> ```



## 4.3、全局配置设置主键生成策略

```yaml
# 加入日志功能
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  # 设置MyBatis-Plus的全局配置
  global-config:
    db-config:
      # 设置实体类所对应的表的统一前缀
      table-prefix: t_
      # 设置统一的主键生成策略
      id-type: auto
```



## 4.4、@TableField



```
经过以上的测试，我们可以发现，MyBatis-Plus在执行SQL语句时，要保证实体类中的属性名和表中的字段名一致 
```



### 4.4.1、情况一

```
若实体类中的属性使用的是驼峰命名风格，而表中的字段使用的是下划线命名风格 
	例如实体类属性userName，表中字段user_name 
此时MyBatis-Plus会自动将下划线命名风格转化为驼峰命名风格 
相当于在MyBatis中配置
```



### 4.4.2、情况二



```
若实体类中的属性和表中的字段不满足情况1 

例如实体类属性name，表中字段username 
此时需要在实体类属性上使用@TableField("username")设置属性所对应的字段名
```



```java
public class User  {
    @TableId(value = "id",type = IdType.AUTO)

    private Long id;
    @TableField(value = "username")
    private String name;
    private Integer age;
    private String email;
}
```





## 4.5、@TableLogic



```java
public @interface TableLogic {

    /**
     * 默认逻辑未删除值（该值可无、会自动获取全局配置）
     */
    String value() default "";

    /**
     * 默认逻辑删除值（该值可无、会自动获取全局配置）
     */
    String delval() default "";
}
```

### 4.5.1、逻辑删除

- 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除的数据 
- **逻辑删除**：假删除，**将对应数据中代表是否被删除字段的状态修改为“被删除状态”**，之后在数据库 中仍旧能看到此条数据记录 

- 使用场景：可以进行数据恢复



### 4.5.2、实现逻辑删除



1. **数据库中创建逻辑删除状态列，设置默认值为0**（0为未删除）
2. 实体类添加逻辑属性

```java
@TableName("user")
public class User  {
    @TableId(value = "id",type = IdType.AUTO)

    private Long id;
    @TableField(value = "username")
    private String name;
    private Integer age;
    private String email;
	@TableLogic
    private Integer isDelete;
}
```

​	3.测试

```
测试删除功能，真正执行的是修改 
UPDATE t_user SET is_deleted=1 WHERE id=? AND is_deleted=0 
测试查询功能，被逻辑删除的数据默认不会被查询 
SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE is_deleted=0
```







# 5、条件构造器和常用接口
