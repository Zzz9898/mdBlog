```
title：springboot整合redis缓存
tags：springboot,redis
grammar_zjwJava
```

- 创建springboot项目，引入依赖(需要在启动类上开启缓存，使用注解@EnableCaching)

  - ```
    		<dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-jpa</artifactId>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.34</version>
            </dependency>
    ```

    注意：springboot选择<version>1.5.6.RELEASE</version>版本，这里先不引入redis，使用默认缓存，操作数据库使用jpa。

  - 配置yml文件

    - ```
      spring:
        datasource:
          driverClassName: com.mysql.jdbc.Driver
          url: jdbc:mysql://localhost:3306/springboot_cache
          username: root
          password: 123
        jpa:
              hibernate:
                ddl-auto: update
              show-sql: true
      ```

    - 其中配置数据库连接信息，jpa的sql打印。

- 创建实体类

  - ```java
    @Entity
    @JsonIgnoreProperties(value = { "hibernateLazyInitializer", "handler" })
    public class Employee{
    
    	@Id
    	@Column(name = "id",nullable = false)
    	private Integer id;
    	private String lastName;
    	private String email;
    	private Integer gender; //性别 1男  0女
    	private Integer dId;
    }
    ```

  - ```java
    @Entity
    @JsonIgnoreProperties(value = { "hibernateLazyInitializer", "handler" })
    public class Department {
    
    	@Id
    	private Integer id;
    	private String departmentName;
    }
    ```

    此处省略getter、setter方法。

- 创建Service、Dao、Controller实现调用数据库

  - Service

    - ```java
      public interface EmployeeService {
          Employee get(int id);
      
          Employee put(Employee employee);
      
          void delete(int id);
      }
      ```

    - ```java
      public interface DepartmentService {
          Department get(int id);
      }
      ```

  - Dao

    - ```java
      @Repository
      public interface EmployeeDao extends JpaSpecificationExecutor<Employee>,JpaRepository<Employee,Integer> {
      
      }
      ```

    - ```java
      @Repository
      public interface DepartmentDao extends JpaRepository<Department,Integer>,JpaSpecificationExecutor<Department> {
      }
      ```

      这里调用基本方法，所以不用自己写方法。

  - Controller

    - ```java
      @RestController
      public class EmployeeController {
      
          @Autowired
          private EmployeeService employeeService;
      
          @GetMapping("/emp/get/{id}")
          public Employee get(@PathVariable int id){
              Employee employee = employeeService.get(id);
              return employee;
          }
      
          @GetMapping("/emp/put")
          public Employee post(Employee employee){
              employeeService.put(employee);
              return employee;
          }
      
          @GetMapping("/emp/delete/{id}")
          public String delete(@PathVariable int id){
              employeeService.delete(id);
              return "success";
          }
      }
      ```
  
    - ```java
      @RestController
      public class DepartmentController {
      
          @Autowired
          private DepartmentService departmentService;
      
          @GetMapping("/dep/get/{id}")
          public Department get(@PathVariable int id){
              Department department = departmentService.get(id);
              return department;
          }
      }
      ```
  
      Controller里就是调用接口中的方法，请求数据。
  
    这里其实就是实现从数据库中查询、修改、删除对应的数据，只是Service层的接口还没有被实现。
  
- Service实现类

  - ```java
    @Service
    public class EmployeeServiceImpl implements EmployeeService {
    
        @Autowired
        private EmployeeDao employeeDao;
    
        @Override
        @Cacheable(value = "emp",condition = "#a0 > 1")
        public Employee get(int id) {
            return employeeDao.getOne(id);
        }
    
        @Override
        @CachePut(value = "emp",key = "#result.id")
        public Employee put(Employee employee) {
            return employeeDao.save(employee);
        }
    
        @Override
        @CacheEvict(value = "emp",beforeInvocation = true)
        public void delete(int id) {
            employeeDao.deleteById(id);
        }
    }
    ```

    - @Cacheable注解，在查询方法上使用，其中value是指缓存组的名称，它是一个数组，里面可以存很多数据，这里没有指定key，那么默认就是传过来的参数，也可以自己指定，condition是指条件，满足该条件就进行缓存，其中的表达式的意思就是当第一个参数大于1的时候，就进行缓存。

      - 此时使用的是默认的缓存，在第一次查询时，会查询数据库，打印出sql语句，此时会将结果作为值，存入缓存，在第二次查询会先查询缓存中的数据，如果有就直接获取到，没有才会查询数据库，

    - @CachePut注解，在更新方法上使用，其中value是指缓存组的名称，这里key设置为返回值的id，如果没有设置，那么将默认用传过来的参数作为key，那么与查询时用id作为key就不一样了，就还会将值存入到缓存中。

      - 此时在调用更新方法后，会更新数据库中的数据，同时将返回值在缓存中更新，所以继续执行查询操作后，由于缓存中存在，而且也已经更新，所以不用查询数据库，也会得到最新的数据。

    - @CacheEvict注解，在删除方法上使用，其中value是指缓存组的名称，beforeInvocation设置true是指在执行方法之前进行缓存中数据的清除，如果在之后删除的方法出错，数据库中数据没有被删除，那么缓存中的数据也是被删除了的，默认为false。

      - 此时在调用删除方法后，会将缓存中的数据也清除掉，如果继续查询，就会从数据库中找数据了。

      注：此处只用了注解一部分属性的值，每个注解还有跟多属性值可以指定实现更多的作用，在有些条件判断，需要用到spel表达式，就是像condition = "#a0 > 1"中 #a0这样的。

- 引入redis

  - 添加依赖

    - ```
      <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
      ```

    - 当引入redis依赖后，spring自动配置类中会发现有redis，则不会继续使用默认的sprin缓存，会自动使用redis缓存。

  - 配置yml文件

    - ```
        redis:
          host: 47.100.91.56
          password: redis
          database: 1
      ```

    - 配置redis服务器地址，密码和使用哪一个数据库。

    此时在进行请求操作后，数据就是存在redis中了，但是在存取对象数据时，可以存成功，但是取出来会反序列化失败，现在开始设置redis配置文件。

- redis配置文件

  - ```java
    @Configuration
    public class RedisConfig {
        @Bean
        public RedisTemplate<Object, Employee> empRedisTemplate(
                RedisConnectionFactory redisConnectionFactory)
                throws UnknownHostException {
            RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
            template.setConnectionFactory(redisConnectionFactory);
            Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
            template.setDefaultSerializer(ser);
            return template;
        }
        @Bean
        public RedisTemplate<Object, Department> deptRedisTemplate(
                RedisConnectionFactory redisConnectionFactory)
                throws UnknownHostException {
            RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
            template.setConnectionFactory(redisConnectionFactory);
            Jackson2JsonRedisSerializer<Department> ser = new Jackson2JsonRedisSerializer<Department>(Department.class);
            template.setDefaultSerializer(ser);
            return template;
        }
    
    
        @Bean
        public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
            RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
            //key多了一个前缀
            //使用前缀，默认会将CacheName作为key的前缀
            cacheManager.setUsePrefix(true);
            return cacheManager;
        }
    
        @Primary
        @Bean
        public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
            RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
            //key多了一个前缀
            //使用前缀，默认会将CacheName作为key的前缀
            cacheManager.setUsePrefix(true);
            return cacheManager;
        }
    
    ```

    这里自定义了两个RedisTemplate，分别是Employee、Department两个实体的，然后分别设置到RedisCacheManager中，这样就可以反序列化对应的实体，需要在注解中指明使用的管理器cacheManager = "employeeCacheManager"，例：@Cacheable(value = "emp",cacheManager = employeeCacheManager")，这个管理器对应的是设置自定义RedisTemplate的方法名，现在存入redis的对象就可以被反序列化出来。

- 使用RedisCacheManager直接操作

  - ```java
    @Service
    public class DepartmentServiceImpl implements DepartmentService {
    
        @Qualifier("deptCacheManager")
        @Autowired
        RedisCacheManager deptCacheManager;
    
        @Autowired
        private DepartmentDao departmentDao;
    
        @Override
        public Department get(int id) {
            Department department = departmentDao.getOne(id);
            Cache dep = deptCacheManager.getCache("dep");
            dep.put("dept:" + id ,department);
            return department;
        }
    }
    ```

    在实现类中注入自定义的RedisCacheManager，在@Qualifier注解中指明使用哪一个缓存管理器，然后用这个缓存管理器可以操作对应实体的数据存入缓存了。

- 注意：

  - 先使用的是springboot 1.5.6.RELEASE版本，该版本设置缓存管理器的配置在sprintboot 2x后有改动，已经不能使用，目前正在试着怎么使用springboot 2x后的版本来配置redis序列化，但是总是抛出错误，不知道是不是jpa的问题还是配置不对，还在慢慢研究，未完待续...

