# 使用mysql
通过mysql-connect-cpp这个包来在c++中使用mysql

## mysql-connect-cpp中xapi使用方法

### Session:会话
Session类是连接到数据库的基础，一切操作都要经过Session完成。

- Session有以下四种构建方法
~~~
Session s1("mysqlx://用户名:密码@主机名:端口号");
Session s2("主机名", 端口号, "用户名", "密码");
Session s3(端口号, "用户名", "密码");
Session s4(
  SessionOption::USER, "用户名",
  SessionOption::PWD, "密码",
  SessionOption::HOST, "主机名",
  SessionOption::PORT, 端口号
);
~~~

### Session在关系型数据库中的操作
xapi提供了两种操控数据库的方法，一种是通过"sql("SQL语句").execute()"直接
用SQL语句操作数据库，另一种是通过其提供的封装函数操作数据库。

#### 1.使用sql语句操作数据库
在创建Session后，可直接通过sql()来输入SQL语句，再通过execute()来执行SQL语句。当我们想要输出结果时，用SqlResult来接受结果通过for循环来输出所有结果。
- 示例代码
~~~cpp
#include <mysqlx/xdevapi.h>
#include <iostream>


using ::std::cout;
using ::std::endl;
using namespace ::mysqlx;

int main(int argc, const char* argv[])
try {
    Session sess("127.0.0.1", 33060, "root", "password");

    // Schema myDb = sess.getSchema("ke");
    sess.sql("USE ke").execute();
    sess.sql("INSERT INTO school VALUES (4,'k',21,'m')").execute();
    SqlResult res = sess.sql("SELECT * FROM school").execute();
    cout << "id   name   age   gender"<< endl;
    for (auto it : res) {
        cout << it[0]<< "   " << it[1]<< "   " << it[2]<< "   " << it [3]<< endl;
    }
}
catch (const mysqlx::Error &err)
{
    cout << "ERROR: " << err << endl;
    return 1;
}
~~~

#### 2.使用封装函数来操作数据库
  

