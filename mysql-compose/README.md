# docker-compose部署MySql
## 1、docker-compose.yml

```yaml
version: '3.7'
services:
  mysql:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=dsz-data-base
    volumes:
      - $PWD/conf:/etc/mysql/conf.d
      - $PWD/logs:/logs
      - $PWD/data:/var/lib/mysql
    container_name: mysql
    ports:
      - "3306:3306"
    restart: always
    command: --default-authentication-plugin=mysql_native_password #这行代码解决无法访问的问题
    
```

## 2、开启mysql远程登录权限

```tex
to see https://www.jb51.net/article/169141.htm
1、进入容器
docker exec -it mysql bash
2、在容器中执行
mysql -u root -p 123456
3、执行以下命令
grant all privileges on *.*  to 'root'@'%' ;(推荐使用)
或者
GRANT ALL PRIVILEGES ON *.*  ‘root'@'%' identified by ‘123123' WITH GRANT OPTION;
4、执行
flush privileges;  刷新权限	
5、Mysql远程连接报错：authentication plugin caching_sha2

mysql 8.0 默认使用 caching_sha2_password 身份验证机制 —— 从原来的 mysql_native_password 更改为 caching_sha2_password。 

从 5.7 升级 8.0 版本的不会改变现有用户的身份验证方法，但新用户会默认使用新的 caching_sha2_password 。
客户端不支持新的加密方式。
方法之一，修改用户的密码和加密方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';
要同赋予用户权限时相同。 若是localhost就同上。%也是如此
mysql8.*的新特性 caching_sha2_password 密码加密方式
以前版本的mysql密码加密使用的是 mysql_native_password
新添加的用户密码默认使用的 caching_sha2_password
如果在以前mysql基础上升级的 就得用户使用的密码加密使用的是 mysql_native_password
如果使用以前的密码加密方式，就修改文件 /etc/my.cnf 
数据库时区问题：
链接数据库时serverTimezone=UTC这个参数出的问题
只要改成serverTimezone=Asia/Shanghai就OK了！
```

