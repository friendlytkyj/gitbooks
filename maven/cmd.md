# 下载指定jar包源码

```
mvn org.apache.maven.plugins:maven-dependency-plugin:3.2.0:get -Dartifact=org.springframework.data:spring-data-mongodb:3.2.3:jar:sources -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true
```