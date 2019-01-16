<project>
  ...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.2</version>
        <configuration>
          <!--
            Exclude JCL and LOG4J since all logging should go through SLF4J.
            Note that we're excluding log4j-<version>.jar but keeping
            log4j-over-slf4j-<version>.jar
          -->
          <packagingExcludes>
            WEB-INF/lib/commons-logging-*.jar,
            %regex[WEB-INF/lib/log4j-(?!over-slf4j).*.jar]
          </packagingExcludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
</project>