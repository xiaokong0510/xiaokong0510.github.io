---
title: 06 使用 JdbcTemplate 访问 MySQL 数据库
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot
  - JDBC 
date: 2021-08-02
description: 连接数据库，从 JdbcTemplate 开始
---

## 1 数据库环境
```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL COMMENT '用户名',
  `phone_number` int(11) DEFAULT NULL COMMENT '手机号',
  `status` bit(1) DEFAULT NULL COMMENT '状态，1 有效；0 无效',
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
## 2 引入依赖

- jdbc
- mysql-connector-java

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

## 3 数据库连接参数

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/mybatis?userUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8
    username: root
    password: root
```
## 4 使用JdbcTemplate操作数据库
1. **实体类**
```java
@Data
@Builder
public class User implements Serializable {
    public static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    private Long id;

    /**
     * 用户名
     */
    private String name;

    /**
     * 手机号码
     */
    private String phoneNumber;

    /**
     * 状态，1 有效；0 无效
     */
    private Byte status;

    /**
     * 创建时间
     */
    private LocalDateTime createTime;

    /**
     * 更新时间
     */
    private LocalDateTime updateTime;
}
```

2. **接口文件定义增删改查方法**

```java
public interface UserService {
    /**
     * 查询所有用户
     *
     * @return 用户集合
     */
    List<User> selectAll();

    /**
     * 根据用户名查询
     *
     * @param id 用户id
     * @return 用户信息
     */
    User selectById(Long id);

    /**
     * 新增用户信息
     *
     * @param model 用户信息
     * @return 主键id
     */
    Long add(User model);

    /**
     * 修改用户信息
     *
     * @param model 用户信息
     * @return 成功 ：1； 失败 ： 0
     */
    int update(User model);


    /**
     * 删除用户
     *
     * @param id 用户id
     * @return 成功 ：1； 失败 ： 0
     */
    int deleteById(Long id);
}
```
3. **使用 JdbcTemplate 实现**

`query(String sql, RowMapper<T> rowMapper)`：其中 rowMapper 对象可以自定义返回结果集的映射

`GeneratedKeyHolder()`：可以获取自增主键的值

```java
public class UserServiceImpl implements UserService {

    private final JdbcTemplate jdbcTemplate;

    UserServiceImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public List<User> selectAll() {
        String sql = "SELECT * FROM `user`";
        return jdbcTemplate.query(sql, (resultSet, rowNum) -> User.builder()
                .id(resultSet.getLong("id"))
                .name(resultSet.getString("name"))
                .phoneNumber(resultSet.getString("phone_number"))
                .status(resultSet.getByte("status"))
                .build());
    }

    @Override
    public User selectById(Long id) {
        String sql = "SELECT * FROM `user` WHERE `id` = ?";
        return jdbcTemplate.queryForObject(sql, (resultSet, rowNum) -> User.builder()
                .id(resultSet.getLong("id"))
                .name(resultSet.getString("name"))
                .phoneNumber(resultSet.getString("phone_number"))
                .status(resultSet.getByte("status"))
                .build(), id);
    }

    @Override
    public Long add(User model) {
        String sql = "INSERT INTO `user` (`name`,`phone_number`,`status`,`create_time`,`update_time`) values (?,?,?,?,?)";
        KeyHolder keyHolder = new GeneratedKeyHolder();
        jdbcTemplate.update(connection -> {
            PreparedStatement psst = connection.prepareStatement(sql, new String[]{"id"});
            psst.setString(1, model.getName());
            psst.setString(2, model.getPhoneNumber());
            psst.setByte(3, model.getStatus());
            psst.setObject(4, model.getCreateTime());
            psst.setObject(5, model.getUpdateTime());
            return psst;
        }, keyHolder);
        return Objects.requireNonNull(keyHolder.getKey()).longValue();
    }

    @Override
    public int update(User model) {
        String sql = "UPDATE `user` SET `name` = ?, `phone_number` = ?,`status` = ? where id =  ? ";
        return jdbcTemplate.update(sql, model.getName(), model.getPhoneNumber(), model.getStatus(), model.getId());
    }

    @Override
    public int deleteById(Long id) {
        String sql = "UPDATE `user` SET `status` = 0 where id =  ? ";
        return jdbcTemplate.update(sql, id);
    }
}
```

4 **编写单元测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class JdbcTemplateApplicationTest {
    @Autowired
    private UserService userService;
    private User user;

    @Before
    public void setUp() {
        this.user = initUser();
    }

    private final static String name = "zhangsan";
    private final static String phoneNumber = "12345678901";

    private User initUser() {
        return User.builder()
                .name(name)
                .phoneNumber(phoneNumber)
                .status((byte) 1)
                .createTime(LocalDateTime.now())
                .updateTime(LocalDateTime.now())
                .build();
    }

    @Test
    @Transactional
    @Rollback
    public void testAddAndSelectAll() {
        Long userId = userService.add(user);
        User user = userService.selectById(userId);
        Assert.assertEquals(phoneNumber, user.getPhoneNumber());
        List<User> users = userService.selectAll();
        Assert.assertEquals(1, users.size());
        Assert.assertEquals(name, users.get(0).getName());
    }

    @Test
    @Transactional
    @Rollback
    public void testAddAndUpdate() {
        Long userId = userService.add(user);
        User user = userService.selectById(userId);
        String newName = "lisi";
        user.setName(newName);
        userService.update(user);
        Assert.assertNotNull(userService.selectById(userId));
    }

    @Test
    @Transactional
    @Rollback
    public void testDelete() {
        Long userId = userService.add(user);
        userService.deleteById(userId);
        User user = userService.selectById(userId);
        Assert.assertEquals(Optional.of((byte) 0).get(), user.getStatus());

    }

}
```

