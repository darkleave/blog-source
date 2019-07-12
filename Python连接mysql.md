---
title: Python 连接mysql数据库
date: 2019-07-12 15:09:42
tags: [python]
---

## pymsql介绍
PyMySQL 是在 Python3.x 版本中用于连接 MySQL 服务器的一个库，Python2中则使用mysqldb。

Django中也可以使用PyMySQL连接MySQL数据库。
[原文章地址](https://www.cnblogs.com/dwenwen/articles/8259638.html)

<!--more-->

## pymsql安装
`pip install pymysql`

登陆验证中,之前学习的都是建立一个文件,通过把用户信息保存在这个文件中,然后一行一行的去读,做校验

```
简单的登陆验证代码

# 获取用户输入
name=input('用户名:')
pwd=input('密码:')
# 校验用户输入的用户名和密码是否正确
with open("userinfo.txt") as f:
    for line in f:
        # 按照空格去分隔每一行,把值分别赋值给name_tmp和pwd_tmp
        name_tmp,pwd_tmp=line.strip().split(" ")
        # 如果能匹配上用户的输入,就表示登陆成功
        if name_tmp ==name and pwd_tmp==pwd:
            print("登陆成功")
            break
    else:
        print('登录失败')
```

上面的代码是通过文件操作来完成登陆验证的,但是有一个问题是每次都要一行一行的去读,而不能像使用数据库那样可以通过索引去找到要验证一条信息

下面引出通过pymsql让Python代码与mysql数据库建立连接

## pymsql的使用
使用方法

```
# 导入pymysql模块
import pymysql
# 连接database
conn = pymysql.connect(host=“你的数据库地址”, user=“用户名”,password=“密码”,database=“数据库名”,charset=“utf8”)
# 得到一个可以执行SQL语句的光标对象
cursor = conn.cursor()
# 定义要执行的SQL语句
sql = """
CREATE TABLE USER1 (
id INT auto_increment PRIMARY KEY ,
name CHAR(10) NOT NULL UNIQUE,
age TINYINT NOT NULL
)ENGINE=innodb DEFAULT CHARSET=utf8;
"""
# 执行SQL语句
cursor.execute(sql)
# 关闭光标对象
cursor.close()
# 关闭数据库连接
conn.close()
```

```
import pymysql
# 校验 用户输入的用户名和密码是否正确
# 去数据库里取数据做判断
# 1.连上数据库,获得的conn是连接
conn = pymysql.connect(host="localhost",database="s8",user="root",password="123456",charset="utf8")
# 只有连接还不行,还需要获取光标,让我能够输入sql语句并执行
cursor=conn.cursor()    #cursor是光标
# 2.执行sql语句
sql="select * from userinfo where id>3;"    #要执行的sql语句写出来
ret = cursor.execute(sql)   #execute是执行的意思,执行sql语句
# 3.关闭光标和连接
cursor.close()
conn.close()

print("%s row in set (0.00 sec)" % ret)

'''
在这个例题中,mysql相当于是serversocket端,pycharm是客户端
客户端需要先于服务端建立连接,也就是本例题中的第1步
建立好连接后需要建立通信,也就是第2步,把我要执行的sql给执行了,并把
执行结果返回给变量ret,并保存在变量中
通信结束后需要断开连接,但在本例题中也获得了光标,需要把光标也关闭
'''

例题
```

sql注入问题

```
import pymysql
name=input('用户名:')
pwd=input('密码:')
# 校验用户输入的用户名和密码是否正确
# 去数据库里取数据做判断
# 1.连接上数据库
conn=pymysql.connect(host='localhost',database='s8',user='root',password='123456',charset='utf8')
# 2.获取光标
cursor=conn.cursor()
# 3.写出要执行的sql语句
sql="select * from userinfo where name= '%s' and pwd='%s';" % (name,pwd)
print(sql)
# 4.执行sql
ret = cursor.execute(sql) #获取影响的行数
# 5.关闭光标和连接
cursor.close()
conn.close()
if ret:
    print('登陆成功')
else:
    print('登陆失败')

'''
这个例子会出现一个问题就是sql注入
就是当用户名上输入alex' or 1=1 -- hehehe,密码不输入也会登陆成功
当我正常输入用户名和密码的时候是可以正常判断执行的
当我在用户名上输入alex'or 1=1 --hehe时,实际在mysql数据库中的显示的sql
语句是select * from userinfo where name= 'alex' or 1=1 -- hehe' and pwd='';
1=1为真 --在mysql中是注释,所以这个sql语句会被当做select * from userinfo where name= 'alex' or 1=1去执行,所以,在没有输入密码的情况下就可以显示登陆成功,这就是sql注入
'''
```

解决sql注入问题

```
# 原来是我们对sql进行字符串拼接
# sql="select * from userinfo where name='%s' and password='%s'" %(user,pwd)
# print(sql)
# res=cursor.execute(sql)

#改写为（execute帮我们做字符串拼接，我们无需且一定不能再为%s加引号了）
sql="select * from userinfo where name=%s and password=%s" #！！！注意%s需要去掉引号，因为pymysql会自动为我们加上
res=cursor.execute(sql,[user,pwd]) #pymysql模块自动帮我们解决sql注入的问题，只要我们按照pymysql的规矩来。
```

```
import pymysql
# 获取用户输入
name= input('用户名:')
pwd =input('密码:')
# 校验用户输入的用户名和密码是否正确
# 去数据库里取数据做判断
# 1.连接上数据库
conn=pymysql.connect(host='localhost',database='s8',user='root',password='123456',charset='utf8')
# 2.获取光标
cursor=conn.cursor()
# 3.要执行的sql语句
sql="select * from userinfo where name='%s' and pwd='%s';"%(name,pwd)
print(sql)
# 4.让pymysql拼接字符串
ret=cursor.execute(sql,[name,pwd])  #获取影响的行数 这里也可以写成ret=cursor.execute(sql,(name,pwd))
# 5.关闭光标和连接
cursor.close()
conn.close()
if ret:
    print('登陆成功')
else:
    print('登陆失败')


'''
第四步中execute中写上要执行的sql语句,要判断的字段,pymsql会自动的去判断输入的信息,
这样就避免了sql注入的问题
'''
```

通过pymsq增加数据库中的数据

```
import pymysql
# 连接
conn=pymysql.connect(host="localhost",user='root',password="123456",charset="utf8",database="s8")
# 获取光标
cursor=conn.cursor()
# 写sql语句
sql="insert into userinfo (name,pwd) VALUE (%s,%s);"
username='egon2'
password='dashabi'

try:
    # 执行sql语句
    cursor.execute(sql,(username,password))
    conn.commit()   #把修改的数据提交到数据库
except Exception as e:
    conn.rollback() #捕捉到错误就回滚

cursor.close()
conn.close()
```

通过pymsql删除数据库中的数据

```
import pymysql
# 连接
conn=pymysql.connect(host="localhost",database="s8",user="root",password="123456",charset="utf8")
# 获取光标
cursor=conn.cursor()
# 写sql语句
sql="delete from userinfo where name=%s;"
username='egon2'
try:
    # 执行sql语句
    cursor.execute(sql,(username,))
    conn.commit()   #把修改提交到数据库
except Exception as e:
    conn.rollback()

cursor.close()
conn.close()
```

通过pymsql修改数据库中的数据

```
import pymysql
# 连接
username=input('用户名')
conn = pymysql.connect(host="localhost", user="root", password="root1234", database="s8", charset="utf8")  # 没有-
# 获取光标
cursor=conn.cursor()
# 写sql语句
sql="update userinfo set pwd=%s where name=%s"
print(sql)

ret=cursor.execute(sql,(username,))
print(ret)
# try:
#     # 执行sql语句
#     cursor.execute(sql,(password,username))
#     conn.commit()   #把修改提交到数据库
# except Exception as e:
#     conn.rollback()
#     print('没有')
cursor.close()
conn.close()
```

通过pymsql查询数据库中的数据

```
import pymysql

# 连接
conn = pymysql.connect(host="localhost", user="root", password="123456", database="s8", charset="utf8")  # 没有-
# 获取光标
cursor = conn.cursor()
# 写sql语句
sql = "select * from userinfo;"
# 执行sql语句
ret=cursor.execute(sql)
print('-->',ret)

# 一次取一条
print(cursor.fetchone())
print(cursor.fetchone())
print(cursor.fetchone())

# 一次取所有
# print(cursor.fetchall())

# 自定义取出多少条
# print(cursor.fetchmany(3))

# 进阶用法
print(cursor.fetchone())
print(cursor.fetchall())
print(cursor.fetchone())    #如果取不到会返回None

# 移动取数据的光标
cursor.scroll(-2)   #默认是相对移动
print(cursor.fetchone())

# 按照绝对位置去移动
cursor.scroll(4,mode='absolute')
print(cursor.fetchone())

cursor.close()
conn.close()
```

通过pymsql批量操作数据库中的数据

```
import pymysql
# 连接
conn=pymysql.connect(host='localhost',database='s8',user='root',password='123456',charset='utf8')
# 获取光标
cursor=conn.cursor()
# 写sql语句
sql="insert into userinfo(name,pwd) values(%s,%s);"
user1='dww1'
pwd1='123456'
user2='dww2'
pwd2='123456'
# 把所有要插入的信息保存在元祖或列表中
data=((user1,pwd1),(user2,pwd2))
try:
    # 执行sql语句
    cursor.executemany(sql,data)    #使用executemany做批量处理
    conn.commit()   #把修改提交到数据库
except Exception as e :
    conn.rollback()
cursor.close()
conn.close()

```

获取插入数据的ID(关联操作时会用到)

```
import pymysql
# 连接
conn=pymysql.connect(host="localhost",user='root',password='123456',database='s8',charset='utf8')
# 获取光标
cursor=conn.cursor()
# 写sql语句
sql="insert into userinfo(name,pwd) values(%s,%s);"
user1='dww'
pwd1='111111'
try:
    # 执行sql语句
    cursor.execute(sql,(user1,pwd1))
    conn.commit()   #把修改提交到数据库
    # 拿到我插入的这条数据的id
    last_id=cursor.lastrowid
    print('--->刚才插入的那条数据的id值是:',last_id)
except Exception as e:
    conn.rollback()
cursor.close()
conn.close()
```

插入数据失败回滚

```
# 导入pymysql模块
import pymysql
# 连接database
conn = pymysql.connect(host=“你的数据库地址”, user=“用户名”,password=“密码”,database=“数据库名”,charset=“utf8”)
# 得到一个可以执行SQL语句的光标对象
cursor = conn.cursor()
sql = "INSERT INTO USER1(name, age) VALUES (%s, %s);"
username = "Alex"
age = 18
try:
    # 执行SQL语句
    cursor.execute(sql, [username, age])
    # 提交事务
    conn.commit()
except Exception as e:
    # 有异常，回滚事务
    conn.rollback()
cursor.close()
conn.close()
```
