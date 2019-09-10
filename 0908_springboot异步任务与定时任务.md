```
title：springboot异步任务和定时任务
tags：springboot,async,scheduled
grammer_zjwJava：true
```

- 创建springboot项目

  - 引入依赖

    - ```
      <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      ```

  - 异步任务

    -  创建controller、service，编写测试方法

      - ```java
        @GetMapping("/get/async")
        public String get(){
            asyncService.getAsync();
            return "success";
        }
        ```

      - ```java
        @Async
        @Override
        public void getAsync() {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("异步处理中...");
        }
        ```

    - 在启动类上开启异步注解，@EnableAsync

      @Async声名这个任务是异步任务，当我们访问get();方法时，如果没有使用异步任务，那么返回结构是要等到getAsync();方法执行完毕，再返回，由于方法中有Thread.sleep(3000);，所以需要等3秒后返回，但是使用异步任务，它并没有等到getAsync();方法执行完毕，而是马上就返回结果了，此时getAsync();还在执行中。

  - 定时任务

    - 创建service，编写测试方法

      - ```
        @Override
        @Scheduled(cron = "0/4 * * * * 0-6")
        public void scheduled() {
        	System.out.println("do...");
        }
        ```

    - 在启动类上开启定时注解，@EnableScheduling

      @Scheduled声名这个任务是定时任务，cron属性是声明定时任务执行的时间，第一个代表秒，第一个代表分，第三个代表时，第四个代表天，第五个代表月，第六个代表星期，具体使用这里不多说明，例如当cron = "0/4 * * * * 0-6"时，那么这个方法便在星期一至星期天每一天里，每隔四秒执行一次，那么启动程序之后，控制台会每个四秒打印一次do...。

