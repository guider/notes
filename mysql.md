#####创建一个用户
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';

##### 赋予新用户权限  否则不能操作数据库
GRANT ALL PRIVILEGES ON * . * TO 'newuser'@'localhost’;