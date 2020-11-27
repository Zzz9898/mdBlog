##### springboot打包成war

- 添加war方式

  ```java
  <packaging>war</packaging>
  ```

- 排除内部依赖

  ```java
  <dependency>
  	<groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
      	<exclusion>
          	<groupId>org.springframework.boot</groupId>
          	<artifactId>spring-boot-starter-tomcat</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>provided</scope>
  </dependency>
  ```

- 引入插件并指定war包名称

  ```java
  <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
  </plugin>
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-war-plugin</artifactId>
      <configuration>
      	<warName>att-upload</warName>
      </configuration>
  </plugin>
  ```

- 启动类中继承SpringBootServletInitializer重写configure

  ```java
  @SpringBootApplication
  public class AttemptFileUploadApplication extends SpringBootServletInitializer {
      public static void main(String[] args) {
          SpringApplication.run(AttemptFileUploadApplication.class);
      }
  
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
          return builder.sources(AttemptFileUploadApplication.class);
      }
  }
  ```

  