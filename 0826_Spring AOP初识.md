---
title: Spring AOP��ʶ
tags: spring,aop
grammar_zjwJava: true
---

�����spring aop���˳�������ʶ����ϴ���򵥽������˽⣬ʵ���˷�����ǿ���ӷ����л�ȡ�������޸Ĳ������ء�
- ����springboot��Ŀ����������
  

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
- �������ʷ���
  

``` 
	@GetMapping("/get/{str}")
    public String getTest(@PathVariable String str,int num){
        System.out.println("�ַ�����:" + str);
        return str;
    }
```

- ����������
  

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
@Aspect����Ϊ�����ࣻ
@Component����spring����
@Pointcut�����е㣬ָ�����ĸ��������룻
@Before�ڷ���ִ��֮ǰ���ã�args(str,num)ָ������Ĳ��������ƶ�Ӧ�����еĲ����������Ի�ȡ�������еĲ���ֵ��
@Around����֪ͨ�����Ի�ȡ����ֵ���޸Ĳ���ֵ���أ���Ҫ����ProceedingJoinPoint�����ö�Ӧ��getArgs()��ȡ���������еĲ�����ע�ⷵ�ص���һ�����飬�����������˳����Ƿ����в��������˳�򣬾ͺñ���֮ǰ�ķ����getArgs()��args[0]��str��ֵ��args[1]��num��ֵ��Ȼ��ʹ��.proceed(args)�������޸Ĳ���ֵ���ء�
   `
