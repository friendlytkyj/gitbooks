# 事务

**MongoDB**从**4.0**开始支持事务，单节点部署不支持事务功能，一定需要副本模式

## 单节点开启副本模式

### 修改mongod.cfg

```
systemLog:
  #destination: file
  #path: D:\ProgramFiles\mongodb-4.2.5\logs\mongod.log
  logAppend: true

storage:
  dbPath: D:\ProgramFiles\mongodb-4.2.5\data

replication:
  # replSetName复制集名称
  replSetName: "rs0"
```

### 添加节点

在命令行连接**MongoDB**，并执行以下命令

`rs.initiate( {_id : "rs0",members: [ { _id: 0, host: "localhost:27017" } ]})`

### 查看副本集状态

执行以下命令

`rs.status()`
