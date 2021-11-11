# 参考文档

`https://www.cnblogs.com/zhoujie/p/pgsql.html`

# 初始化data目录

`pg_ctl -D D:\ProgramFiles\PostgreSQL\13\data\ initdb`

# 修改postgresql.conf

找到`- Where to Log -`

```
logging_collector = on
log_directory = 'D:/ProgramFiles/PostgreSQL/13/logs'
log_rotation_size = 100MB
```

# 启动

`pg_ctl -D D:\ProgramFiles\PostgreSQL\13\data start`

# 停止

`pg_ctl -D D:\ProgramFiles\PostgreSQL\13\data stop`

# 查看状态

`pg_ctl -D D:\ProgramFiles\PostgreSQL\13\data status`

# 查看版本

`pg_ctl -V`

# 创建数据库用户

`createuser -P test_user`

# 创建数据库/密码

`createdb -O test_user -E UTF8 -e test`
