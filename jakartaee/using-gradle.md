* Open a command prompt and type
  ```bash
  gradle init
  ```
* Select type of project to generate => 2. application
* Select implementation language => 3. java
* Split functionality across multiple subprojects => no - only one application project
* Select build script DSL => Groovy
* Generate build using new APIs and behaviour => no
* Select Test framework => JUnit Jupiter
* Enter the project name
* Enter source package
* Go to the gradle tomcat plugin [page](https://github.com/bmuschko/gradle-tomcat-plugin "bmuschko")
* Add the jar pluging in build.gradle
  ```groovy
  buildscript {
      repositories {
          gradlePluginPortal()
      }

      dependencies {
          classpath 'com.bmuschko:gradle-tomcat-plugin:2.7.0'
      }
  }
  ```
  Place this block before the plugin{} block.
* Now apply the tomcat plugin. To get start quickly use this block
  ```groovy
  apply plugin: 'war'
  apply plugin: 'com.bmuschko.tomcat'
  ```
* Add the tomcat library in dependencies
  ```groovy
  def tomcatVersion = '9.0.1'
  tomcat "org.apache.tomcat.embed:tomcat-embed-core:${tomcatVersion}",
          "org.apache.tomcat.embed:tomcat-embed-logging-juli:9.0.0.M6",
          "org.apache.tomcat.embed:tomcat-embed-jasper:${tomcatVersion}"
  ```
* The full build.gradle is
  ```groovy
  buildscript {
      repositories {
          gradlePluginPortal()
      }

      dependencies {
          classpath 'com.bmuschko:gradle-tomcat-plugin:2.7.0'
      }
  }

  plugins {
      id 'application'
  }

  apply plugin: 'com.bmuschko.tomcat'


  repositories {
      mavenCentral()
  }

  dependencies {
      testImplementation 'org.junit.jupiter:junit-jupiter:5.9.1'

      implementation 'com.google.guava:guava:31.1-jre'
      implementation 'jakarta.servlet:jakarta.servlet-api:6.0.0'

      def tomcatVersion = '9.0.1'
      tomcat "org.apache.tomcat.embed:tomcat-embed-core:${tomcatVersion}",
              "org.apache.tomcat.embed:tomcat-embed-logging-juli:9.0.0.M6",
              "org.apache.tomcat.embed:tomcat-embed-jasper:${tomcatVersion}"
  }

  tomcat {
      httpProtocol = 'org.apache.coyote.http11.Http11Nio2Protocol'
      ajpProtocol  = 'org.apache.coyote.ajp.AjpNio2Protocol'
  }

  application {
      mainClass = 'com.test.App'
  }

  tasks.named('test') {
      useJUnitPlatform()
  }
  ```
* Create a bean class, a simple POJO under the java->package
  ```java
  package com.test;

  public class Message {
      public String getContent() {
          return "Hello world";
      }
  }
  ```
* Create the folder 'webapp' under app->src->main
* Create a new JSP file 'hello.jsp' under this 'webapp' folder
* Run the gradle command
  ```bash
  ./gradlew tomcatRun
  ```
  Here, tomcatRun is a standard command provided by bmuschko.
* Test in the browser
  [page](http://localhost:8080/app/hello.jsp "bmuschko")
