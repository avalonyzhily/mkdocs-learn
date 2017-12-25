### jar包安装到本地
```
mvn install:install-file -Dfile=D:\thrift-0.9.2.jar -DgroupId=org.apache.thrift -DartifactId=libthrift -Dversion=0.9.2 -Dpackaging=jar 
```

### jar包推送到私服
```
mvn deploy:deploy-file -DgroupId=org.apache.thrift -DartifactId=libthrift -Dversion=1.12 -Dpackaging=jar -Dfile=D:\thrift-0.9.2.jar -Durl=http://ip:port/nexus/content/repositories/thirdparty/ -DrepositoryId=thirdparty 
```