---
layout: default
title: Useful Plug-ins
date: 18.11.2019
---

# Useful Plug-ins

| Plugin                    |                                                          |
| ------------------------- | -------------------------------------------------------- |
| Sonarlint                 | detect and fix quality issues as you write code.         |
| Checkstyle                | write Java code that adheres to a coding standard        |
| Enhenced class decompiler | Class decompiler                                         |
| Bytecode outline          | Class decompiler                                         |
| Egit                      | Eclipse Team provider for the Git version control system |



## Maven Enforcer Plugin

Checks this [list](https://maven.apache.org/enforcer/enforcer-rules/index.html) . Added to project pom like this;

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-enforcer-plugin</artifactId>
   <configuration>
      <rules>
         <bannedDependencies>
            <excludes>
               <exclude>junit:junit</exclude>
               <exclude>junit:junit-dep</exclude>
            </excludes>
         </bannedDependencies>
         <dependencyConvergence />
      </rules>
   </configuration>
   <executions>
      <execution>
         <goals>
            <goal>enforce</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

Usage:  `mvn verify` or `mvn enforcer:enforce` 

## OWASP Dependency-Check

Checks Common Vulnerabilities for used jars. Added to project pom like this;

```xml
<plugin>
   <groupId>org.owasp</groupId>
   <artifactId>dependency-check-maven</artifactId>
   <version>5.2.2</version>
   <configuration>
      <suppressionFiles>
         <suppressionFile>owasp-suppressions.xml</suppressionFile>
      </suppressionFiles>
      <failBuildOnCVSS>8</failBuildOnCVSS>
      <assemblyAnalyzerEnabled>false</assemblyAnalyzerEnabled>
      <failOnError>true</failOnError>
   </configuration>
   <executions>
      <execution>
         <goals>
            <goal>check</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

Usage:  `mvn verify` or `mvn enforcer:enforce` 

## SpotBugs Maven Plugin

A  static code analysis tool. Added to project pom like this;

```xml
<plugin>
   <groupId>com.github.spotbugs</groupId>
   <artifactId>spotbugs-maven-plugin</artifactId>
   <version>3.1.12.2</version>
   <dependencies>
      <dependency>
         <groupId>com.github.spotbugs</groupId>
         <artifactId>spotbugs</artifactId>
         <version>4.0.0-beta4</version>
      </dependency>
   </dependencies>
</plugin>
```

Usage:  `mvn spotbugs:check`  and for GUI version  `mvn spotbugs:gui` 



