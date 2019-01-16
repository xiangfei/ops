Special Variables
basedir	The directory that the current project resides in.
project.baseUri	The directory that the current project resides in, represented as an URI. Since Maven 2.1.0
maven.build.timestamp	The timestamp that denotes the start of the build. Since Maven 2.1.0-M1
The format of the build timestamp can be customized by declaring the property maven.build.timestamp.format as shown in the example below:


<project>
  ...
  <properties>
    <maven.build.timestamp.format>yyyyMMdd-HHmm</maven.build.timestamp.format>
  </properties>
  ...
</project>
${projcet.maven.build.timestamp.format}