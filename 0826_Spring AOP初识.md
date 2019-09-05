---
title: Spring AOP初识
tags: spring,aop
grammar_zjwJava: true
---

今天对spring aop做了初步的认识，结合代码简单进行了了解，实现了方法增强、从方法中获取参数、修改参数返回。
- 创建springboot项目，引入依赖
  

``` 
	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
     </dependency>
```
- 创建访问方法
  

``` 
	@GetMapping("/get/{str}")
    public String getTest(@PathVariable String str,int num){
        System.out.println("字符串是:" + str);
        return str;
    }
```

- 创建切面类
  

``` 
	@Aspect
	@Component
	public class DemoAop {

    @Pointcut("execution(public * com.sx.dome.controller.DemoController.getTest(..))")
    public void aopDemo(){

    }

    @Before("aopDemo() && args(str,num)")
    public void doBefore(JoinPoint joinPoint, String str, int num){
        System.out.println("doBefore----------" + str + ", " + num);
    }

    @After("aopDemo() && args(str,num)")
    public void doAfter(JoinPoint joinPoint, String str, int num){
        System.out.println("doAfter----------" + str + ", " + num);
    }

    @Around("aopDemo()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("around------------");
        Object[] args = joinPoint.getArgs();
        args[0] = String.valueOf(args[1]);
        Object proceed = joinPoint.proceed(args);
        return proceed;
    }
}
```
@Aspect声明为切面类；
@Component交给spring管理；
@Pointcut声明切点，指明在哪个方法切入；
@Before在方法执行之前调用，args(str,num)指明传入的参数，名称对应方法中的参数名，可以获取到方法中的参数值；
@Around环绕通知，可以获取参数值后修改参数值返回，需要传入ProceedingJoinPoint，调用对应的getArgs()获取方法中所有的参数，注意返回的是一个数组，数组里参数的顺序就是方法中参数定义的顺序，就好比在之前的方法里，getArgs()后，args[0]是str的值，args[1]是num的值，然后使用.proceed(args)方法，修改参数值返回。
   `
