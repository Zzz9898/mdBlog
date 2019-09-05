---
title: springboot整合shiro
tags: springboot,shiro
grammar_zjwJava: true
---

在网上看了好多springboot整合shiro的博客，然后自己就试着也做了一个demo。

- ##### 创建springboot项目，引入依赖

  - ```java
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
    
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
    </dependency>
    
    <dependency>
         <groupId>org.projectlombok</groupId>
         <artifactId>lombok</artifactId>
    </dependency>
    ```

    此处数据库操作使用jpa来完成。

- 创建实体类

  - ```java
    Data
    @Entity
    @JsonIgnoreProperties(value = { "hibernateLazyInitializer", "handler" })
    public class User implements Serializable {
    
        @Id
        private long id;
    
        private String username;
    
        private String password;
    
        private int admin;
    
        @ManyToMany
        @JoinTable(name = "user_role_rel",joinColumns = @JoinColumn(name = "userId"),inverseJoinColumns = @JoinColumn(name = "roleId"),foreignKey = @ForeignKey(name = "null"))
        private List<Role> roles;
    }
    ```

    用户表，@JsonIgnoreProperties(value = { "hibernateLazyInitializer", "handler" })这个注解是因为如果不加上的话会出现：

    No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer

    这个错误信息。

    此处使用了@ManyToMany多对多注解，一个用户有多个角色，一个角色有多个用户，在数据库中有一个中间表，在做查询时默认带出角色信息。

  - ```java
    @Data
    @Entity
    public class Role implements Serializable {
    
        @Id
        private long id;
    
        private String name;
    
        @ManyToMany
        @JoinTable(name = "role_permission_rel",joinColumns = @JoinColumn(name = "roleId"),inverseJoinColumns = @JoinColumn(name = "permissionId"),foreignKey = @ForeignKey(name = "null"))
        private List<Permission> permissions;
    }
    ```

    角色表，此处使用了@ManyToMany多对多注解，一个角色有多个权限，一个权限有多个角色，在数据库中有一个中间表，在做查询时默认带出权限信息。

  - ```java
    @Data
    @Entity
    public class Permission implements Serializable {
    
        @Id
        private long id;
    
        private String name;
    
        private String presource;
    }
    ```

    权限表。

- 创建service与dao类

  - ```java
    public interface UserService {
        User get(long id);
    
        User findByUserName(String username);
    }
    ```

  - ```java
    @Service("UserService")
    @Transactional
    public class UserServiceImpl implements UserService {
    
        @Autowired
        private UserDao userDao;
    
        @Override
        public User get(long id) {
            return userDao.getOne(id);
        }
    
        @Override
        public User findByUserName(String username) {
            return userDao.findByUsername(username);
        }
    }
    ```

  - ```java
    @Repository("UserDao")
    public interface UserDao extends JpaRepository<User,Long>,JpaSpecificationExecutor<User> {
        User findByUsername(String username);
    }
    ```

    此处只简单的进行实现通过名称和id来查找用户信息两个方法。

- 创建shiro配置文件

  - ```java
    @Configuration
    @Component
    public class ShiroConfig {
    
        @Bean
        public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
            ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
            // 必须设置 SecurityManager
            shiroFilterFactoryBean.setSecurityManager(securityManager);
            // setLoginUrl 如果不设置值，默认会自动寻找Web工程根目录下的"/login.jsp"页面 或 "/login" 映射
            shiroFilterFactoryBean.setLoginUrl("/notLogin");
            // 设置无权限时跳转的 url;
            shiroFilterFactoryBean.setUnauthorizedUrl("/notRole");
            // 设置拦截器
            Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
    //        //游客，开发权限
    //        filterChainDefinitionMap.put("/guest/**", "anon");
    //        //用户，需要角色权限 “user”
    //        filterChainDefinitionMap.put("/user/**", "roles[user]");
    //        //管理员，需要角色权限 “admin”
    //        filterChainDefinitionMap.put("/admin/**", "roles[admin]");
            //开放登陆接口
            filterChainDefinitionMap.put("/login", "anon");
            //其余接口一律拦截
            //主要这行代码必须放在所有权限设置的最后，不然会导致所有 url 都被拦截
            filterChainDefinitionMap.put("/**", "authc");
    
            shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
            System.out.println("Shiro拦截器工厂类注入成功");
            return shiroFilterFactoryBean;
        }
    
        /**
         * 凭证匹配器
         * （由于我们的密码校验交给Shiro的SimpleAuthenticationInfo进行处理了
         * ）
         *
         * @return
         */
        @Bean
        public HashedCredentialsMatcher hashedCredentialsMatcher() {
            HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
            hashedCredentialsMatcher.setHashAlgorithmName("md5");//散列算法:这里使用MD5算法;
            hashedCredentialsMatcher.setHashIterations(1);//散列的次数，比如散列两次，相当于 md5(md5(""));
            return hashedCredentialsMatcher;
        }
    
        /**
         * 注入 securityManager
         */
        @Bean
        public SecurityManager securityManager() {
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            // 设置realm.
            securityManager.setRealm(shiroRealm());
            return securityManager;
        }
    
        /**
         * 自定义身份认证 realm;
         * <p>
         * 必须写这个类，并加上 @Bean 注解，目的是注入 ShiroRealm，
         * 否则会影响 ShiroRealm 中其他类的依赖注入
         */
        @Bean
        public ShiroRealm shiroRealm() {
            ShiroRealm shiroRealm = new ShiroRealm();
            shiroRealm.setCredentialsMatcher(hashedCredentialsMatcher());
            return shiroRealm;
        }
    
        /**
         * 开启shiro aop注解支持.
         * 使用代理方式;所以需要开启代码支持;
         *
         * @param securityManager
         * @return
         */
        @Bean
        public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
            AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
            authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
            return authorizationAttributeSourceAdvisor;
        }
    ```

    shirFilter();方法，创建shiro中拦截器，其中anno时不需要授权，登陆就可以访问，authc是需要登陆授权才能访问。

    hashedCredentialsMatcher();方法，配置加密规则。

    securityManager();方法，注入securityManager，设置自定义realm。

    shiroRealm();方法，注入自定义realm，设置加密规则。

    authorizationAttributeSourceAdvisor();方法，开启AOP注解支持，可以使用注解进行权限控制。

    此处只简单的配置了以上几点，还有更多可以配置的。

- 创建自定义realm

  - ```java
    @Component
    public class ShiroRealm extends AuthorizingRealm {
    
        @Autowired
        private UserService userService;
    
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
            //User user = (User) principalCollection.getPrimaryPrincipal();
            String primaryPrincipal = (String) principalCollection.getPrimaryPrincipal();
            User user = userService.findByUserName(primaryPrincipal);
            for (Role role : user.getRoles()) {
                simpleAuthorizationInfo.addRole(role.getName());
                for (Permission permission : role.getPermissions()) {
                    simpleAuthorizationInfo.addStringPermission(permission.getPresource());
                }
            }
            return simpleAuthorizationInfo;
        }
    
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            String username = (String) authenticationToken.getPrincipal();
            User user = userService.findByUserName(username);
            if(user == null){
                return null;
            }
            SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(user.getUsername(),
                    user.getPassword(),
    //                ByteSource.Util.bytes(user.getUsername()),
                    this.getName());
            return simpleAuthenticationInfo;
        }
    }
    ```

    其中需要继承AuthorizingRealm实现认证doGetAuthenticationInfo();和授权doGetAuthorizationInfo();两个方法，userService就是提供从数据库获取数据，例如通过用户名查找用户信息，通过用户名查找对应的用户的角色和拥有的权限。

    认证：从(String) authenticationToken.getPrincipal();获取用户名，通过用户名查找数据库中对应的用户信息，然后将信息作为simpleAuthenticationInfo的参数返回，第一个参数为用户名或者用户对象，第二个参数为用户的密码，shiro会自己判断密码是否与输入的一致，第三个参数为盐，就是密码是加盐处理的话，这个就要传入对应的盐，这个参数可有可无，最后一个参数就是当前realm的名称。

    授权：从(String) principalCollection.getPrimaryPrincipal();获取用户名信息，通过用户名查找用户信息，然后遍历用户拥有的角色和权限作为参数添加到simpleAuthenticationInfo对象中，然后返回该对象。

- 创建controller类

  - ```java
    @RestController
    public class UserController {
    
        @Autowired
        private UserService userService;
    
        @GetMapping("get/{id}")
        public User get(@PathVariable long id){
            User user = userService.get(id);
            return user;
        }
    
        @RequiresPermissions("user:add")
        @PostMapping("/post")
        public String post(@RequestParam("username") String username,
                         @RequestParam("password") String password){
            System.out.println(username+ " " +password);
            return null;
        }
    
        @PostMapping("/login")
        public String login(@RequestParam("username") String username,
                            @RequestParam("password") String password){
            UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(username, password);
            Subject subject = SecurityUtils.getSubject();
            try {
                subject.login(usernamePasswordToken);
            }catch (IncorrectCredentialsException ice){
                return "密码不正确";
            }catch(UnknownAccountException uae){
                return "账号不存在";
            }catch(AuthenticationException ae){
                return "状态不正常";
            }
            if(subject.isAuthenticated()){
                System.out.println("认证成功");
                return "shiro: login success";
            }else{
                return "shiro: login fail";
            }
        }
    
        @GetMapping("/logout")
        public String logout(){
            Subject subject = SecurityUtils.getSubject();
            subject.logout();
            return "logout success";
        }
    }
    ```

    提供通过id获取用户信息get();方法、添加用户信息方法post();(这个方法只做演示，没有处理)、

    登录方法login();、登出方法logout();。

    之前在配置文件中设置了.put("/login", "anon");，所以访问登录方法时不被拦截，当未登录访问需要拦截的方法时，会跳到.setLoginUrl("/notLogin");中/notLogin这个路径。

    登录方法：将传过来的用户名和密码信息存到new UsernamePasswordToken(username, password);

    这个对象中，创建主体SecurityUtils.getSubject();对象，使用主体subject.login(usernamePasswordToken);来认证信息，捕获对应的异常进行信息说明，如果登录成功shiro会将信息加入到缓存中，供之后的操作使用。

    登录后访问get();方法，由于这个方法有设置对应的权限访问，只要登录后就可以访问，所以可以成功访问，但是访问post();方法，由于该在方法上使用@RequiresPermissions("user:add")注解，只有拥有user:add的权限才可以访问，而当前登录的用户没有该权限，所以程序抛出异常：org.apache.shiro.authz.AuthorizationException: Not authorized to invoke method: public java.lang.String com.shiro.demo.controller.UserController.post(java.lang.String,java.lang.String)

    说没有这个授权去执行这个方法，前端返回信息：

    "message": "Subject does not have permission [user:add]",说主体没有这个user:add权限，即实现了shiro权限认证。