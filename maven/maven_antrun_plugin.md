<plugin>
 2             <groupId>org.apache.maven.plugins</groupId>
 3             <artifactId>maven-antrun-plugin</artifactId>
 4             <version>1.6</version>
 5             <executions>
 6               <execution>
 7                 <id>compile</id>
 8                 <phase>compile</phase>
 9                 <configuration>
10                   <target>
11                     <echo message="********************copy profile propertie file *************************"/>                                                                                                                                                                                    
12                     <copy file="src/main/resources/config/common-product.properties"
13                           tofile="${buildDirectory}/classes/common.properties" overwrite="true"/>
14                   </target>
15                 </configuration>
16                 <goals>
17                   <goal>run</goal>
18                 </goals>
19               </execution>
20             </executions>
21         </plugin>
phase  compile 编译时运行， or install  安装完成运行 提示phase 运行不需要先运行plugin mvn antrun:copy (something like this)