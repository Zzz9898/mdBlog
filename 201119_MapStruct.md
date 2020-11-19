###### MapStruct

- 引入依赖(需要结合lombok)

  ```java
  <dependency>
      <groupId>org.mapstruct</groupId>
      <artifactId>mapstruct</artifactId>
      <version>1.3.1.Final</version>
  </dependency>
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
  </dependency>
  ```

- 插件引入

  ```java
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
          <source>1.8</source> <!-- depending on your project -->
          <target>1.8</target> <!-- depending on your project -->
          <annotationProcessorPaths>
              <path>
                  <groupId>org.mapstruct</groupId>
                  <artifactId>mapstruct-processor</artifactId>
                  <version>1.3.1.Final</version>
              </path>
              <path>
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
              </path>
          </annotationProcessorPaths>
      </configuration>
  </plugin>
  ```

- 创建DTO、DO、枚举

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class PersonDTO {
      private String userName;
  
      private Integer age;
  
      private String birthday;
  
      private Gender gender;
  
      private String fixed;
  
  }
  
  -----
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class PersonDO {
      private Integer id;
  
      private String name;
  
      private int age;
  
      private LocalDateTime birthday;
  
      private Integer gender;
  
      private String fixed;
  
  }
  
  -----
  public enum Gender {
      MAN(1, "男"),WOMAN(2, "女"),UNKNOWN(3, "不详");
  
      private Integer value;
      private String desc;
  
      Gender(int value, String desc) {
          this.value = value;
          this.desc = desc;
      }
      ...省略setter、getter
  }
  ```

- 创建Converter

  ```java
  @Mapper
  public interface PersonConverter {
  
      PersonConverter INSTANCE = Mappers.getMapper(PersonConverter.class);
  
      @Mapping(source = "name", target = "userName") // 名称不对应
      @Mapping(target = "fixed", constant = "fixed") // 赋固定值
      @Mapping(target = "birthday",dateFormat = "yyyy-MM-dd HH:mm:ss") //时间转换
      PersonDTO do2dto(PersonDO personDO);
  
      /**
       * 自定义基本类型转成枚举
       * 注：一般情况下，对于以下情况可以做自动类型转换：
       * 1. 基本类型及其他们对应的包装类型。
       * 2. 基本类型的包装类型和String类型之间
       * 3. String类型和枚举类型之间
       * @param value
       * @return
       */
      default Gender integerToGender(int value){
          for (Gender gender : Gender.values()) {
              if (gender.getValue().equals(value)){
                  return gender;
              }
          }
          return null;
      }
  }
  ```

- 测试

  ```java
  public class MapStructDemoMain {
      public static void main(String[] args) {
          PersonDO personDO = new PersonDO(1, "do", 21, LocalDateTime.now(), Gender.MAN.getValue(), "position");
          PersonDTO personDTO = PersonConverter.INSTANCE.do2dto(personDO);
          System.out.println(personDTO);
          /* 控制台:
            PersonDTO(userName=do, age=21, birthday=2020-11-19 16:32:08, gender=MAN, fixed=fixed)
           */
      }
  }
  ```

  