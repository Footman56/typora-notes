# 一、概念

```
JDBC是Java DataBase Connectivity的缩写，它是Java程序访问数据库的标准接口。
使用Java程序访问数据库时，Java代码并不是直接通过TCP连接去访问数据库，而是通过JDBC接口来访问，而JDBC接口则通过JDBC驱动来实现真正对数据库的访问。

可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序
```

<img src="/Users/peilizhi/screenshots/截屏2021-08-30 23.42.26.png" style="zoom:50%;" />

通过第三方数据库厂商提供的驱动来操作数据库

# 二、组件

## 2.1 Driver

```
装载驱动程序
Driver接口由数据库厂家提供，在编程中要连接数据库，必须先装载特定厂商的数据库驱动程序，不同的数据库有不同的装载方法

装载MySql驱动：Class.forName("com.mysql.jdbc.Driver");

装载Oracle驱动：Class.forName("oracle.jdbc.driver.OracleDriver");
```

## 2.2 Connection接口

```
Connection与特定数据库的连接（会话），在连接上下文中执行sql语句并返回结果。 可以理解成管道，与数据库进行操作

DriverManager.getConnection(url, user, password)方法建立在JDBC URL中定义的数据库Connection连接上。

不同的数据连接url不同
连接MySql数据库：Connection conn = DriverManager.getConnection("jdbc:mysql://host:port/database", "user", "password");

连接Oracle数据库：Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@host:port:database", "user", "password");
```

```
接口方法：
createStatement()：创建向数据库发送sql的statement对象。
prepareStatement(sql) ：创建向数据库发送预编译sql的PrepareSatement对象。
prepareCall(sql)：创建执行存储过程的callableStatement对象。
setAutoCommit(boolean autoCommit)：设置事务是否自动提交。
commit() ：在链接上提交事务。
rollback() ：在此链接上回滚事务。
```

## 2.3 Statement 接口

```
用于执行静态SQL语句并返回它所生成结果的对象. 

继承类：
Statement：由createStatement创建，用于发送简单的SQL语句（不带参数）。不能保证sql安全
PreparedStatement ：继承自Statement接口，由preparedStatement创建，用于发送含有一个或多个参数的SQL语句。PreparedStatement对象比Statement对象的效率更高，并且可以防止SQL注入，所以我们一般都使用PreparedStatement。
CallableStatement：继承自PreparedStatement接口，由方法prepareCall创建，用于调用存储过程。
```

```
接口方法：
execute(String sql):运行语句，返回是否有结果集
executeQuery(String sql)：运行select语句，返回ResultSet结果集。
executeUpdate(String sql)：运行insert/update/delete操作，返回更新的行数。
addBatch(String sql) ：把多条sql语句放到一个批处理中。
executeBatch()：向数据库发送一批sql语句执行。
```

## 2.4  ResultSet接口

```
执行的结果。
getString(int index)、getString(String columnName)：获得在数据库里是varchar、char等类型的数据对象。
getFloat(int index)、getFloat(String columnName)：获得在数据库里是Float类型的数据对象。
getDate(int index)、getDate(String columnName)：获得在数据库里是Date类型的数据。
getBoolean(int index)、getBoolean(String columnName)：获得在数据库里是Boolean类型的数据。
getObject(int index)、getObject(String columnName)：获取在数据库里任意类型的数据。

ResultSet还提供了对结果集进行滚动的方法：

next()：移动到下一行
Previous()：移动到前一行
absolute(int row)：移动到指定行
beforeFirst()：移动resultSet的最前面。
afterLast() ：移动到resultSet的最后面。
```

关闭连接的顺序： **ResultSet->Statement->Connection**

# 三、执行流程

```
1、加载驱动
2、获取链接
3、获取statement
4、编写sql
5、执行Sql,获取结果集
6、遍历结果集，组装对象
7、有事务就提交事务，没有就关闭连接
```

```java
public static void main(String[] args) throws SQLException {
        // 加载驱动
        DriverManager.registerDriver(new Driver());
        // 获取连接
        final Connection connection = DriverManager.getConnection("jdbc:mysql://47.96.235.128:3306/student?useSSL=false&serverTimezone=UTC", "huochai", "Admin123@");
        // 获取statement
        final Statement statement = connection.createStatement();
        // 编写sql
        String sql="select * from sys_student";
        // 执行sql
        final ResultSet resultSet = statement.executeQuery(sql);
        // 遍历结果集
        while (resultSet.next()){
            final int id = resultSet.getInt("id");
            final String name = resultSet.getString("name");
            final int age = resultSet.getInt("age");
            final String sex = resultSet.getString("sex");
            System.out.println("id = " + id + "\tname = " + name + "\tage = " + age + "\tsex = " + sex);
        }
        // 关闭资源
        resultSet.close();
        statement.close();
        connection.close();
    }
```

**要防止驱动被多次加载，可以将注册驱动的部分放在静态代码块中**

# 四、statement 安全

**Statement执行，其实是拼接sql语句的。 先拼接sql语句，然后在一起执行**

```
 因为sql是拼凑出来了，所以可以在sql中加入一些绝对的判断来避免检查
```

```
PrepareStatement
相比较以前的statement， 预先处理给定的sql语句，对其执行语法检查。 在sql语句里面使用 ? 占位符来替代后续要传递进来的变量。 后面进来的变量值，将会被看成是字符串，不会产生任何的关键字。

执行方法：
String sql = "insert into t_user values(null , ? , ?)";
ps = conn.prepareStatement(sql);
 //给占位符赋值 从左到右数过来，1 代表第一个问号， 永远是1开始。
 ps.setString(1, userName);
 ps.setString(2, password);
```

```
(1) 使用PreparedStatement，代码的可读性和可维护性比Statement高。
(2) PreparedStatement 能最大可能提高性能。
DBServer会对预编译语句提供性能优化。因为预编译语句有可能被重复调用，所以语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。
在statement语句中,即使是相同操作但因为数据内容不一样，所以整个语句本身不能匹配，没有缓存语句的意义。事实是没有数据库会对普通语句编译后的执行代码缓存。这样每执行一次都要对传入的语句编译一次。
(3) PreparedStatement能保证安全性，但 Statement有sql注入等安全问题。
```

# 五、事务

```
为确保数据库中数据的一致性,数据的操纵应当是离散的成组的逻辑单元:当它全部完成时,数据的一致性可以保持,而当这个单元中的一部分操作失败,整个事务应全部视为错误,所有从起始点以后的操作应全部回退到开始状态。

事务的操作:先定义开始一个事务,然后对数据作修改操作,这时如果提交(COMMIT),这些修改就永久地保存下来,如果回退(ROLLBACK),数据库管理系统将放弃您所作的所有修改而回到开始事务时的状态。
```

# 六、批处理

**Mysql 不支持批处理**

```
当需要批量插入或者更新记录时。可以采用Java的批量更新机制，这一机制允许多条语句一次性提交给数据库批量处理。
多条sql 一起交给数据库执行

DBC的批量处理语句包括下面两个方法：

addBatch(String)：添加需要批量处理的SQL语句或是参数；
executeBatch（）；执行批量处理语句；
```

```
Statement sm = conn.createStatement();
sm.addBatch(sql1);
sm.addBatch(sql2);
...
//批量处理
sm.executeBatch()
//清除sm中积攒的参数列表
sm.clearBatch();

将多条sql 一起提交
```

```
preparedStatement ps = conn.preparedStatement(sql);
for(int i=1;i<100000;i++){
    ps.setInt(1, i);
    ps.setString(2, "name"+i);
    ps.setString(3, "email"+i);
    ps.addBatch();
    if((i+1)%1000==0){
        //批量处理
        ps.executeBatch();
        //清空ps中积攒的sql
        ps.clearBatch();
    }
}

多条sql 参数不同，一起提交
```

**批量处理应该设置一个上限，当批量处理列表中的sql累积到一定数量后，就应该执行，并在执行完成后，清空批量列表。**

# 七、元数据

```
Java 通过JDBC获得连接以后，得到一个Connection 对象，可以从这个对象获得有关数据库管理系统的各种信息，包括数据库中的各个表，表中的各个列，数据类型，触发器，存储过程等各方面的信息。
```

```
获取这些信息的方法都是在DatabaseMetaData类的对象上实现的，而DataBaseMetaData对象是在Connection对象上获得的。
 获取的方法 final DatabaseMetaData metaData = connection.getMetaData();
getURL()：返回一个String类对象，代表数据库的URL。
getUserName()：返回连接当前数据库管理系统的用户名。
isReadOnly()：返回一个boolean值，指示数据库是否只允许读操作。
getDatabaseProductName()：返回数据库的产品名称。
getDatabaseProductVersion()：返回数据库的版本号。
getDriverName()：返回驱动驱动程序的名称。
getDriverVersion()：返回驱动程序的版本号。
```

```
ResultSetMetaData
可用于获取关于 ResultSet 对象中列的类型和属性信息的对象：

getColumnName(int column)：获取指定列的名称
getColumnCount()：返回当前 ResultSet 对象中的列数。
getColumnTypeName(int column)：检索指定列的数据库特定的类型名称。
getColumnDisplaySize(int column)：指示指定列的最大标准宽度，以字符为单位。
isNullable(int column)：指示指定列中的值是否可以为 null。
isAutoIncrement(int column)：指示是否自动为指定列进行编号，这样这些列仍然是只读的。
```

# 八、数据库连接池

```
普通的JDBC数据库连接使用 DriverManager 来获取，每次向数据库建立连接的时候都要将 Connection 加载到内存中，再验证用户名和密码。需要数据库连接的时候，就向数据库要求一个，执行完成后再断开连接。这样的方式将会消耗大量的资源和时间。

数据库连接池的基本思想就是为数据库连接建立一个“缓冲池”。预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕之后再放回去。

数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中，这些数据库连接的数量是由最小数据库连接数来设定的。无论这些数据库连接是否被使用，连接池都将一直保证至少拥有这么多的连接数量。连接池的最大数据库连接数量限定了这个连接池能占有的最大连接数，当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。
```

