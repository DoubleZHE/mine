## 把jar包添加到本地仓库

```
mvn install:install-file -DgroupId= -DartifactId= -Dversion= -Dpackaging=jar -Dfile=
```

`DgroupId`：表示jar对应的groupId



`DartifactId`: 表示jar对应的artifactId



`Dversion`: 表示jar对应的 version



`Dfile`：表示jar包所在的路径名


> mvn install:install-file -DgroupId='mysql' -DartifactId='mysql-connector-java' -Dversion='5.1.7' -Dpackaging='jar' -Dfile='mysql-connector-java-5.1.7-bin.jar'