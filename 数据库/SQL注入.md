SQL注入
-----
1.
    SQL 注入页也是一种常见的 Web攻击方式，主要是利用后端程序的漏洞，针对数据库 （主要是关系型数据库）进行攻击。攻击者通常通过输入精心构造的参数，绕过服务器端的验证，来执行恶意 SQL 语句，SQL注入会造成拖库（原始数据表被攻击者获取），绕过权限验证，或者串改、破坏、删除数据。数据往往是一个网站甚至一个公司的生命，一但SQL漏洞被攻击者利用通常会产生非常严重的后果，对于SQL 注入的防范需要开发者倍加重视。

    下面通过一个例子来演示SQL注入的攻击过程，对于一个登陆验证的请求一般需要通过用户输入的用户名和密码进行查询验证，查询SQL 语句如下：
    SELECT * FROM users WHERE username = "$username" AND password = "$password";

    比如攻击者已知一个用户名archer2017，可以构造密码为 anywords" OR 1=1，此时，此时后端程序的执行的SQL语句为：
    SELECT * FROM users WHERE username = "archer2017" AND password = "anywords" OR 1=1;

    这样无论输入什么样的密码，都会要绕过验证，如果更验证一些，构造密码为 anywords" OR 1=1;DROP TABLE users ，那么整个表都将被删除，执行SQL如下:
    SELECT * FROM users WHERE username = "archer2017" AND password = "anywords" OR 1=1;DROP TABLE users;

2.SQL 注入的防范
    SQL 注入发生绝大多数情况都是直接使用户输入参数拼装 SQL 语句造成的，防范 SQL 注入，主要的方式丢失避免直接使用用户输入的数据。
    使用预编译语句（PreparedStatement）：一方面可以加速 SQL的执行，一方面可以
防止SQL注入。

    SQL语句的编译分为如下几个阶段：
    基本解析：包括SQL语句的语法、语义解析，以及对应的表和列是否存在等等。
    编译：将SQL语句编译成机器理解客理解的中间代码格式。
    查询优化：编译器在所有的执行方案中选择一个最优的。
    缓存：缓存优化后的执行方案。
    执行阶段：执行最终查询方案并返回给用户数据。

    预编译语句指的是在缓存之后，在执行阶段的之前的编译后的语句，通过占位符来替代查询查询参数。同样的SQL，如果参数不同普通的SQL语句每次请求都会进行编译，而预编译语句只会编译一次，在执行阶段会从缓存中取出预编译语句并将占位符替换成查询参数数据，而在这个阶段SQL 语句已经是编译后的语句，参数数据只能最为纯数据使用，不能作为SQL语句的一部分，通过SQL字符串的拼装已经不起作用，所以可以避免SQL注入。

    Java 代码中使用预编译语句：
    String query = "SELECT * FROM users WHERE username = ? AND password = ?";
    PreparedStatement pstmt = connection.prepareStatement( query );
    pstmt.setString( 1, username); 
    pstmt.setString( 2, password); 
    ResultSet results = pstmt.executeQuery( );



tags:数据库

