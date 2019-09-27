```
title：springboot整合spring security
tags：springboot,spring security
grammar_zjwJava：true
```

- 创建springboot项目，引入依赖

  - ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    <version>5.1.34</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-springsecurity5</artifactId>
        <version>3.0.4.RELEASE</version>
    </dependency>
    ```

    其中，引入数据库连接，操作数据库使用jpa，引入thymeleaf、security和springsecurity5，springboot版本使用的是<version>2.1.3.RELEASE</version>。

  - 配置yml文件

    - spring:
        thymeleaf:
          mode: HTML5
        datasource:
          driverClassName: com.mysql.jdbc.Driver
          url: jdbc:mysql://localhost:3306/spring_securitypro
          username: root
          password: 123
        jpa:
              hibernate:
                ddl-auto: update
              show-sql: true

- 静态页面

  ![0916_1](.\images\0916_1.png)

  pages文件夹下的每个不同的level需要的角色不同。

- 创建实体类

  - User

    - ```java
      @Entity
      public class User{
      
          @Id
          private Long id;
      
          private String username;
      
          private String password;
      
          @ManyToMany
          @JoinTable(name = "user_role_rel",joinColumns = @JoinColumn(name = "userId"),
                  inverseJoinColumns = @JoinColumn(name = "roleId"),foreignKey = @ForeignKey(name = "none"))
          private List<Role> roles = new ArrayList<>();
      }
      ```

      使用jpa中@ManyToMany注解关联Role，@JoinTable表示使用中间表进行关联。

  - Role

    - ```java
      @Entity
      public class Role {
      
          @Id
          private Long id;
      
          private String name;
      }
      ```

    这里都省略了setter、getter方法。

- 创建UserDao

  - ```java
    @Repository
    public interface UserDao extends JpaSpecificationExecutor<User>,JpaRepository<User,Long> {
        User findByUsername(String s);
    }
    ```

    这里实现一个根据用户名查询对应用户的方法，由于用户表关联了角色表，所以查询会将用户拥有的角色一同查询出来。

- 编写配置类

  - ```java
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
        @Bean
        public UserDetailsServiceImpl userDetailService (){
            return new UserDetailsServiceImpl () ;
        }
    
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            //super.configure(http);
            http.authorizeRequests().antMatchers("/").permitAll()
                    .antMatchers("/level2/**").hasRole("VIP2")
                    .antMatchers("/level3/**").hasRole("VIP3");
            http.formLogin().usernameParameter("user").passwordParameter("pwd")
                    .loginPage("/userlogin");
            http.rememberMe().rememberMeParameter("remeber");
            http.logout().logoutSuccessUrl("/");
        }
    
        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            auth.userDetailsService(userDetailService ())
                    .passwordEncoder(new MyPasswordEncoder());
        }
    }
    ```

    注意，@EnableWebSecurity注解一定要加上，UserDetailsServiceImpl这个类我们待会再创建，其中重写configure(HttpSecurity http);方法，authorizeRequests()配置请求的url，antMatchers("/").permitAll()表示该路径不需要权限就可以访问，antMatchers("/level2/**").hasRole("VIP2")表示/level2/下的请求需要拥有角色VIP2才可以访问。http.formLogin()开启登录表单，usernameParameter("user").passwordParameter("pwd")表示表单中登录账号和密码参数的名称，如果使用默认页面则不需要设置，这里使用了自定义页面，loginPage("/userlogin")对应的页面请求路径。rememberMe().rememberMeParameter("remeber");设置使用记住我的功能，其中参数为选择框的名称。http.logout().logoutSuccessUrl("/");设置注销后跳转的url请求地址。

    configure(AuthenticationManagerBuilder auth);方法设置了自定义userDetailService和密码加密规则，其中userDetailService ();方法就是获得我们需要创建的UserDetailsServiceImpl类，MyPasswordEncoder这个类待会也要创建。

- 创建密码加密规则配置类

  - ```java
    public class MyPasswordEncoder implements PasswordEncoder {
        @Override
        public String encode(CharSequence charSequence) {
            return charSequence.toString();
        }
    
        @Override
        public boolean matches(CharSequence charSequence, String s) {
            return s.equals(charSequence.toString());
        }
    }
    ```

    此处设置的是密码不进行加密，直接明文验证。

- 创建UserDetailsServiceImpl，实现userDetailService

  - ```java
    @Service("UserDetailsServiceImpl")
    @Primary
    public class UserDetailsServiceImpl implements UserDetailsService {
        @Autowired
        private UserDao userDao;
    
        @Transactional
        @Override
        public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
            com.sx.demo.entity.User user = userDao.findByUsername(s);
            List<GrantedAuthority> grantedAuthorityList = new ArrayList<>() ;
            if(user != null){
                for (Role role : user.getRoles()) {
                    grantedAuthorityList.add(new SimpleGrantedAuthority(role.getName()));
                }
            }
            return new org.springframework.security.core.userdetails.User(s,user.getPassword(),grantedAuthorityList);
        }
    }
    ```

    不知道为什么需要设置@Primary，不然会报错需要你设置@Primary，好像是说有多个实现类，不知道那个是默认使用的，所以需要设置@Primary，表示这个为默认实现类。

    实现UserDetailsService后，重写loadUserByUsername方法，其中调用userDao中的勇敢用户名查找用户的方法，但是这里的User与security中的User重名了，使用全路径引入。List<GrantedAuthority>保存User中拥有的角色信息，每次保存new SimpleGrantedAuthority类传入角色的名称，注意，在数据库中如果一个角色是admin或者其他，都需要在前加上ROLE_，因为security会默认在请求时加上这个前缀，然后返回一个

    security中的User对象，传入用户名，密码和权限列表。

- 创建Controller

  - ```
    @Controller
    public class KungfuController {
        private final String PREFIX = "pages/";
    
        @GetMapping("/")
        public String index() {
            return "welcome";
        }
    
        @GetMapping("/userlogin")
        public String loginPage() {
            return PREFIX+"login";
        }
    
        @PreAuthorize("hasAnyRole('VIP1')")
        @GetMapping("/level1/{path}")
        public String level1(@PathVariable("path")String path) {
            return PREFIX+"level1/"+path;
        }
    
        @GetMapping("/level2/{path}")
        public String level2(@PathVariable("path")String path) {
            return PREFIX+"level2/"+path;
        }
    
        @GetMapping("/level3/{path}")
        public String level3(@PathVariable("path")String path) {
            return PREFIX+"level3/"+path;
        }
    ```

    这里实现请求的路径跳转的页面，由于之前设置了/level2和/level3下的请求都需要对应的角色，而也可以使用注解 @PreAuthorize("hasAnyRole('VIP1')")设置该路径下的请求需要对应的角色，这样就实现了对不同角色对不同请求的访问。

- 页面标签说明

  - ```html
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org"
    	  xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
    <h1 align="center">欢迎光临武林秘籍管理系统</h1>
    <div sec:authorize="!isAuthenticated()">
    	<h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
    </div>
    <div sec:authorize="isAuthenticated()">
    	<h2><span sec:authentication="name"></span>，您好,您的角色有：
    		<span sec:authentication="principal.authorities"></span></h2>
    	<form th:action="@{/logout}" method="post">
    		<input type="submit" value="注销"/>
    	</form>
    </div>
    <hr>
    <div sec:authorize="hasRole('VIP1')">
    	<h3>普通武功秘籍</h3>
    	<ul>
    		<li><a th:href="@{/level1/1}">罗汉拳</a></li>
    		<li><a th:href="@{/level1/2}">武当长拳</a></li>
    		<li><a th:href="@{/level1/3}">全真剑法</a></li>
    	</ul>
    </div>
    <div sec:authorize="hasRole('VIP2')">
    	<h3>高级武功秘籍</h3>
    	<ul>
    		<li><a th:href="@{/level2/1}">太极拳</a></li>
    		<li><a th:href="@{/level2/2}">七伤拳</a></li>
    		<li><a th:href="@{/level2/3}">梯云纵</a></li>
    	</ul>
    </div>
    <div sec:authorize="hasRole('VIP3')">
    	<h3>绝世武功秘籍</h3>
    	<ul>
    		<li><a th:href="@{/level3/1}">葵花宝典</a></li>
    		<li><a th:href="@{/level3/2}">龟派气功</a></li>
    		<li><a th:href="@{/level3/3}">独孤九剑</a></li>
    	</ul>
    </div>
    </body>
    </html>
    ```

    引入了xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4"，其中sec:authorize="!isAuthenticated()"表示不需要认证就可以访问的元素，sec:authorize="isAuthenticated()"反之就是需要认证才可以访问的元素，sec:authorize="hasRole('VIP1')"就是需要有VIP1这个角色才可以访问的元素，这样可以实现不同角色所显示的页面不同的效果。