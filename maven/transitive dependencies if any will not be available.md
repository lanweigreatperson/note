#### 错误解决：transitive dependencies if any will not be available

之前在打包的时候出出现这样的错误，今天又出现了记录下解决

当执行命令：`mvn dependency:tree -Dverbose` 查看依赖树时发现有个warning:
Invalid POM for mypackage:projA2:jar:1.0, transitive dependencies (if any) will not be available, enable debug logging for more details.
执行命令`mvn dependency:tree -X`  查看详细信息，有ERROR提示需要填入版本号。
解决方法：在最顶层的父pom目录下执行：`mvn clean install -Dmaven.test.skip`
需要更新修改的父pom到本地才行。