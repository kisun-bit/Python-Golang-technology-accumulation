
14.2 配置管理
===========================

PostgreSQL支持远程访问
>>>>>>>>>>>>>>>>>>>>>>>>>>

1. 使用 `find / -name postgresql.conf`  找到 `postgresql.conf`
2. 在最后添加用户参数：

 * `listen_address = ‘*’`，注意不要被注释掉

3. 启用密码验证 `#password_encryption = on` 修改为 `password_encryption = on`
4. 修改 `pg_hba.conf` 文件的内容：

 * 可访问的用户 `ip` 段
 * 在文件末尾加入：`host  all  all  0.0.0.0/0  md5`

5. 重启 `postgreSQL`  数据库：`systemctl restart postgresql`