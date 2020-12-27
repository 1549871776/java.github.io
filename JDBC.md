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