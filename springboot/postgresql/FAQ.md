# PostgreSQL

## 启动时提示"不是一个数据库集群目录"

运行启动命令`pg_ctl start -D D:\ProgramFiles\PostgreSQL\13\data`
出现如下提示:
`pg_ctl: 目录 "D:/ProgramFiles/PostgreSQL/13/data"不是一个数据库集群目录`

执行`pg_ctl -D D:\ProgramFiles\PostgreSQL\13\data\ initdb`

## 启动时提示"Permission denied"

could not open log file "D:/ProgramFiles/PostgreSQL/13/logs/postgresql-2021-11-04_151249.log": Permission denied