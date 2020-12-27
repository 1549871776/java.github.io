# JAVA JDBC
## 步骤
1. 创建java工程
2. 在工程下面新建lib文件夹
3. 将jar驱动包mysql-connector-java-8.0.21.jar放入lib文件夹中，右键文件夹选择Add as Library...选项，单击OK
4. 倘若引入驱动包后，程序无法连接到数据库，可能情况为驱动包版本落后，下载最新的驱动包重新导入，配置好后可能会报时区错误  
Failed to execute goal org.mybatis.generator:mybatis-generator-maven-plugin:1.3.5:generate (default-cli) on project songci-serv: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support. -> [Help 1]  
- 解决方法：  
mysql> set global time_zone='+8:00';
Query OK, 0 rows affected  
Navicat中命令列介面执行即可。或者在数据库连接配置中加上serverTimezone=GMT%2B8（代表东八区），如下：  
String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8";

## 代码
1. 创建测试函数  
public void test01() throws Exception{...  
在函数上方添加单元测试符@Test，出现报错情况，按Alt+Enter键，选择Add'JUnit4'to classpath选项，单击OK

2. 注册驱动  
DriverManager.registerDriver(new Driver());
3. 获取连接对象，填写数据库的名字，登录数据库服务器的用户名和密码
String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8";  
String username = "root";  
String password = "zch13776564595";
4. 连接数据库  
 Connection connection = DriverManager.getConnection(url, username, password);
5. 创建执行SQL语句的对象statement  
Statement stm = connection.createStatement();
6. 选择数据表  
String sql = "select * from t_user";
7. 执行SQL语句，执行查询语句调用stm对象的executeQuery方法，获取结果集resultSet对象  
ResultSet rst = stm.executeQuery(sql);  
rst可以理解为数据库里的每一条数据  
如果是执行查询的SQL语句，则调用executeQuery()方法返回值是查询到的结果集，如果是执行增删改的SQL语句，则调用executeUpdate()返回值是int类型，表示收到影响的行数，它还有一个execute()方法，既能执行增删改，又能执行查询，但是无法返回查询的结果集  

8. 将rst结果中的数据值打印出来
```
while(rst.next()){
            //每遍历一次，就获取一行数据
            //获取该行的每个字段的数据，根据字段名获取
            int id = rst.getInt("uid");
            String name = rst.getString("uname");
            String pwd  = rst.getString("pwd");

            System.out.println(id + ":" + name + ":" + pwd);
        }
```
9. 关闭资源，后创建的先关闭  
rst.close();  
stm.close();  
connection.close();  

## 完整代码

```
package com.itheima;

import com.mysql.jdbc.Driver;
import org.junit.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class TestJDBC {
    @Test
    public void test01() throws Exception{
        //目标:使用JDBC执行SQL语句，查询user表中的所有数据，并打印出来
        //1.注册驱动
        DriverManager.registerDriver(new Driver());
        //2.获得连接对象
        String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8";
        String username = "root";
        String password = "zch13776564595";
        Connection connection = DriverManager.getConnection(url, username, password);

        //3.创建执行SQL语句的对象statement
        Statement stm = connection.createStatement();

        //编写SQL语句，
        String sql = "select * from t_user";

        //4.执行SQL语句，执行查询语句调用stm对象的executeQuery方法，获取结果集resultSet对象
        ResultSet rst = stm.executeQuery(sql);

        //5.遍历出结果集中的结果
        while(rst.next()){
            //每遍历一次，就获取一行数据
            //获取该行的每个字段的数据，根据字段名获取
            int id = rst.getInt("uid");
            String name = rst.getString("uname");
            String pwd  = rst.getString("pwd");

            System.out.println(id + ":" + name + ":" + pwd);
        }

        //6.关闭资源，后创建的先关闭
        rst.close();
        stm.close();
        connection.close();
    }
}
```


## 实体Bean的编写要求
实体Bean一般在src文件夹下创建一个pojo包，在包里创建一个User类，该类用来封装数据，其编写要求为：  
1. 成员变量私有
2. 私有成员变量要具备公有的get和set方法
3. 重写toString方法，为了打印该对象的数据
4. 一定要提供无参构造（因为以后的框架都需要它的无参构造）
5. 如果属性是基本数据类型，那么声明他的包装类型（int默认值为0，integer默认数据为null）
6. 最好是让实体Bean类，去实现Serializable接口

```
package com.itheima.pojo;

public class User implements Serializable{
    private Integer id;
    private String username;
    private String password;


    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    public Integer getId() {
        return this.id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```


## 配置文件resources
1. 在项目文件中创建resources文件，右键选择Mark Directory as-> 选择Resources root（实际上它和java类为同级目录，但是便于管理）  
2. 在resources文件中新建一个文件，起名为jdbc.properties-> 编写配置信息
```
jdbc.username = root
jdbc.password = 123
jdbc.url = jdbc:mysql:///test?serverTimezone=GMT%2B8
jdbc.driverClassName = com.mysql.jdbc.Driver
```
3. 在Bean的User类中修改静态变量，读取配置文件（在注册驱动前）
```
            //1. 将配置文件转换成字节输入流,使用类加载器
            InputStream is = JDBCUtil.class.getClassLoader().getResourceAsStream("jdbc.properties");
            //2. 创建一个Properties对象
            Properties properties = new Properties();
            //3. properties对象加载配置文件
            properties.load(is);
            //4. 调用properties方法获取值
            username = properties.getProperty("jdbc.username");
            password = properties.getProperty("jdbc.password");
            url = properties.getProperty("jdbc.url");
```
补充：使用ResourceBundle读取jdbc.properties文件中的内容
```
@Test
    public void testResourceBundle() {
        //目标:使用ResourceBundle读取jdbc.properties文件中的内容
        ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
        String username = bundle.getString("jdbc.uname");
    }

```


## 用户登录案例
```
    public static void main(String[] args) throws SQLException {
        // 登录案例
        // 1. 控制台提示：输入用户名
        System.out.println("请输入用户名");
        // 2. 获取用户输入的用户名
        Scanner scanner = new Scanner(System.in);
        String username = scanner.nextLine();
        // 3. 控制台提示输入密码
        System.out.println("请输入密码");
        // 4. 获取用户输入的密码
        String password = scanner.nextLine();
        // 5. 校验用户名和密码是否正确
        // 以username和password作为条件，到user表中查询
        // 使用JDBC执行查询的SQL语句
        Connection connection = JDBCUtil.getConnection();
        Statement stm = connection.createStatement();
        String sql = "select * from t_user where uname = '"+ username +"' and pwd = '"+ password +"'";
        ResultSet rst = stm.executeQuery(sql);

        // 如果结果集中有数据，就说明用户名和密码正确
        if(rst.next()) {
            //登陆成功
            System.out.println("登陆成功...");
        }else {
            System.out.println("登陆失败...");
        }
        JDBCUtil.close(rst, stm, connection);
    }
```
当sql语句拼接 or '='时，会导致数据库取出所有数据，此时无论用户名输入任意值都能登陆成功，这种情况叫做SQL的注入，一定要避免此情况的出现！！！
使用prepareStatement进行预编译可以解决

## prepareStatement 
```
 @Test
    public void test07() throws SQLException {
        // 使用prepareStatement执行查询语句：根据id查询用户信息，查询id为3的用户信息
        // 1.获取连接
        Connection connection = JDBCUtil.getConnection();
        int id = 3;
        // 2.对要执行的SQL语句进行预编译，需要传入数据的地方，使用？做占位符
        String sql = "select * from t_user where uid = ?";
        // 预编译
        PreparedStatement pstm = connection.prepareStatement(sql);
        // 3.给占位符位置进行赋值
        pstm.setInt(1, id);

        // 4. 执行SQL语句
        ResultSet rst = pstm.executeQuery();

        // 5.遍历结果集，获取结果
        while (rst.next()) {
            int userId = rst.getInt("uid");
            String username = rst.getString("uname");
            String pwd = rst.getString("pwd");

            User user = new User();
            user.setId(userId);
            user.setUsername(username);
            user.setPassword(pwd);

            System.out.println(user);
        }
        JDBCUtil.close(rst, pstm, connection);
    }
```

## 转账案例
```
package com.itheima.utils;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;


/**
 * 使用事务：逻辑上的一组操作单元，要么同时执行成功，要么同时执行失败，它的操作必须具备原子性
 * 具体的事务使用步骤：
 *      1. 在执行这一组逻辑操作之前，要开启事务
 *          start transaction
 *          设置不自动提交事务：connection.setAutoCommit(false)
 *      2. 在执行完这组逻辑操作，没有出现任何异常的情况下，提交事务:connection.commit()
 *      3. 在执行这组逻辑操作的过程中，如果出现异常，则回滚事务:connection.rollback()
 *
 *  什么场景下会使用到事务
 *  1. 转账
 *  2. 批量删除，批量添加
 *  3. 面面项目添加试题：添加题目信息（往question表添加数据）、添加题目选项信息（往question_item表中添加数据）
 */
public class TestTransfer {
    public static void main(String[] args) throws SQLException {
        Connection connection = null;
        PreparedStatement pstm = null;
        PreparedStatement pstm2 = null;
        try {
            // 目标：zs向ls转账520元
            // 1.获取链接
            connection = JDBCUtil.getConnection();
            // 开启事务
            connection.setAutoCommit(false);
            // 第一步：zs扣款520元
            String sql = "update account set money = money - ? where name = ?";
            // 预编译SQL语句
            pstm = connection.prepareStatement(sql);
            // 设置问号处的值
            pstm.setDouble(1, 520);
            pstm.setString(2, "zs");
            // 执行SQL语句
            pstm.executeUpdate();

            // 第二步：ls收款520元
            String sql2 = "update account set money = money + ? where name = ?";
            pstm2 = connection.prepareStatement(sql2);
            pstm2.setDouble(1, 520);
            pstm2.setString(2, "ls");
            pstm2.executeUpdate();

            // 提交事务
            connection.commit();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
            connection.rollback();
        }


        pstm.close();
        pstm2.close();
        connection.close();
    }
}
```

## 今日总结

一、常用函数
1. if相关的函数(重要)
   if()
   ifnull()
   
2. 字符串相关的函数(了解)
   1. 字符窜拼接
   2. 大小写转换
   3. 去掉左右空格
   4. 截取字符串
   
3. 日期时间相关的函数(了解)
   1. current_date()
   2. current_time()
   3. now()
   
4. 数学相关的函数(了解)
  1.绝对值
  2.向上取整
  3.向下取整
  4.幂
  5.随机数
  
二、JDBC
1. JDBC的思想(接口的思想、解耦的思想)

2. 使用JDBC执行SQL语句的步骤:(只要掌握变化的就可以了
   1. 注册驱动: Class.forName("com.mysql.jdbc.Driver");  掌握为什么要使用这种方式注册驱动
   2. 获得连接对象: DriverManager.getConnection(url,username,password);
      url 数据库的连接地址: jdbc:mysql:///数据库名
	  username 数据库的用户名
	  password 数据库密码
	  
   3. 创建Statement对象，该对象可以执行SQL语句
      Statement stm = conn.createStatement();
	  
   4. 使用statement对象执行SQL语句
      1. executeUpdate(sql);执行增删改的语句，返回值是int类型，表示受到影响的行数
      2. executeQuery(sql);执行查询的SQL语句，返回值是ResultSet，查询到的结果集
	  
   5. 如果有结果集，则遍历结果集，使用while循环遍历
   6. 关闭资源，后创建的先关闭
   
3. 使用POJO对象封装查询到的结果
   1. 如果查询到的是一行数据，则封装到一个POJO对象中
   2. 如果查询到的是多行数据，那么每行数据封装到一个POJO对象中，所有的POJO对象存储到List集合中
   
4. POJO对象的编写规范:
   1. 成员变量私有
   2. 给所有的私有成员变量提供公有的get和set方法
   3. 必须有无参构造
   4. 建议重写toString方法，便于打印
   5. 属性如果是基本类型，声明成为对应的包装类型
   6. 建议实现Serializable接口
   
5. JDBC工具类的封装
   1. 静态代码块中注册驱动
   2. getConnection()方法获取连接对象，返回值是Connection对象
   3. close()方法关闭资源，需要关闭的资源就作为方法的参数传入
      方法重载:
		1. 俩参数的Statement stm,Connection conn
		2. 仨参数的ResultSet rst,Statement stm,Connection conn
		
   4. 使用配置文件解决硬编码问题(主要要去理解什么是硬编码问题)
   
6. 登录案例
   1. 提示用户输入用户名
   2. 获取用户输入的用户名
   3. 提示用户输入密码
   4. 获取用户输入的密码
   5. 通过执行SQL语句，校验用户名和密码
   6. 输出登录结果
   
   注意点: 有SQL注入问题
   
7. preparedStatement
   1. 作用:预编译SQL语句，提升SQL语句的执行效率，可以防止发生SQL注入问题
   2. 使用步骤:
	  1. 编写SQL语句，如果要传入参数则用?占位符代替
	  2. 调用connection对象的prepareStatement(sql);进行预编译
	  3. 如果SQL语句中有?占位符，则需要给?占位符处设置参数
	  4. 调用preparedStatement的方法执行SQL语句
	     1. executeUpdate()执行增删改的SQL语句
	     2. executeQuery()执行查询的SQL语句
		 
8. 使用preparedStatement 优化登录案例，防止SQL注入

9. JDBC操作事务
   前提: 这组逻辑操作使用的是同一个connection
   1. 开启事务
      1. 什么时候: 那一组逻辑操作开始之前
	  2. 怎么开启事务: connection.setAutoCommit(false)
   2. 提交事务
      1. 什么时候提交: 执行完这组逻辑操作，没有出现异常
	  2. 怎么提交事务: connection.commit()
   3. 回滚事务
      1. 什么时候回滚: 执行这组逻辑操作的时候，出现异常，则在catch代码块中回滚
	  2. 怎么回滚事务: connection.rollback()

