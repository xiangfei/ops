<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.xf.demo</groupId>
    <artifactId>mavenweb</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>Java EE 6 webapp project</name>


    <properties>
        <!-- Explicitly declaring the source encoding eliminates the following 
            message: -->
        <!-- [WARNING] Using platform encoding (UTF-8 actually) to copy filtered 
            resources, i.e. build is platform dependent! -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!-- Define the version of JBoss' Java EE 6 APIs we want to import. Any 
            dependencies from org.jboss.spec will have their version defined by this 
            BOM -->
        <javaee6.web.spec.version>2.0.0.Final</javaee6.web.spec.version>
        <!-- Alternatively, comment out the above line, and un-comment the line 
            below to use version 3.0.0.Beta1-redhat-1 which is a release certified to 
            work with JBoss EAP 6. It requires you have access to the JBoss EAP 6 maven 
            repository. -->
        <!-- <javaee6.web.spec.version>3.0.0.Beta1-redhat-1</javaee6.web.spec.version> -->
    </properties>


    <build>
        <directory>${project.basedir}/target</directory>
        <outputDirectory>${project.build.directory}/classes</outputDirectory>
        <finalName>${project.artifactId}-${project.version}</finalName>
        <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>

        <sourceDirectory>${project.basedir}/src</sourceDirectory>
        <resources>
            <resource>
                <directory>${project.basedir}/resources</directory>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <webappDirectory>${project.basedir}/WebRoot</webappDirectory>

                    <!-- webResources> <resource> <targetPath>WEB-INF/classes</targetPath> 
                        <directory>${project.basedir}/resources</directory> </resource> </webResources> -->
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
needs to do others ,but the architype is ok