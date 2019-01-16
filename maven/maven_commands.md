常用命令：

创建空项目:mvn archetype:create -DgroupId=com.banger -DartifactId=myapp

根据pom文件产生eclipse项目文件    mvn eclipse:eclipse

删除maven生成的项目文件:mvn eclipse:clean

编译 :mvn compile

打包: mvn package

发布到本地资源库: mvn install

发布到远程资源库:  mvn deploy(需要配置远程发布方式)

清除编译临时文件：mvn clean

测试：mvn test

打出一个所有依赖包都存在的打包: mvn assembly:assembly

打源码包(依赖也能指定下载源码，可以使调试开发更加方便): mvn source:jar


使maven2在下载依赖包的同时下载其源代码包的方法：

1. 使用maven命令：mvn dependency:sources 下载依赖包的源代码。

2. 使用参数： -DdownloadSources=true 下载源代码jar。 -DdownloadJavadocs=true 下载javadoc包。





Maven打war包命令和步骤：
创建一个web工程并且带有maven的pom.xml文件。
1:mvn archetype:create -DgroupId=com.wj.test -DartifactId=test-website -DpackageName=com.wj.test.maven.website -DarchetypeArtifactId=maven-archetype-webapp -Dversion=1.0

创建maven项目,根据maven的默认的格式.<maven2>
mvn org.apache.maven.plugins:maven-archetype-plugin:2.0-alpha-5:generate

创建maven项目,根据maven的默认的格式.<maven3>
mvn archetype:generate

格式化这个项目,变成maven在eclipse中的格式。
2:mvn eclipse:eclipse

注意:maven主源码默认放在 src/main/java中
整理target目录。也就是重新编译项目到target目录
3:mvn clean compile

注意:maven测试代码默认放在 src/test/java中
测试代码
4:mvn clean test

把当前打的包放在Maven本地仓库中
5:mvn clean install

查看当前项目的已解析依赖关系
6:mvn dependency:list

查看当前项目的依赖树
7:mvn dependency:tree
 

打war包
8:mvn clean package

--指定资源文件进行替换
mvn clean -Dmaven.test.skip=true install  -Dfilter.file="../../parentPoj/resources/deploy-137.properties"

mvn -Dmaven.test.skip=true deploy -Dfilter.file="../../parentPoj/resources/deploy-137.properties"

--查看项目包依赖关系
mvn dependency:tree


----------------------------------------------------

Maven打jar包命令和步骤：
创建一个web工程并且带有maven的pom.xml文件。
1:mvn archetype:create -DgroupId=com.newsocoft.test -DartifactId=test-core -DpackageName=com.newsocoft.test.maven.core -Dversion=1.0

格式化这个项目,变成maven在eclipse中的格式。
2:mvn eclipse:eclipse

打war包
3:mvn package


mvn site:site
mvn site就可以为项目生成一系列可以用来描述项目信息的网页，maven2中的一大部分插件就是专门在这时候发挥效用的，
它们可以根据项目的结构，源代码，测试，SCM信息等，生成各种特殊功能的报表


查看插件参数
mvn help:describe -Dplugin=插件动作名字