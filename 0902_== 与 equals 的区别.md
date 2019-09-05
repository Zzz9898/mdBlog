```
title：== 与 equals 的区别
tags：string,equals
grammar_zjwJava: true
```

区别：

1. “==” 比较的是两个引用在内存中指向的是不是同一对象（即同一内存空间），也就是说在内存空间中的存储位置是否一致，如果两个对象的引用相同时（指向同一对象时），“==”操作符返回true，否则返回false。

2.  equals比较的是值是否相等，只要值相等，就返回true，否则返回false。



```java
String str1 = "String";
String str2 = "String";
String str3 = new String("String");
System.out.println(str1 == str2);
System.out.println(str1.equals(str3));
System.out.println(str2 == str3);
```

结果：

```
true
true
false
```

其中str1与str2都是指向”String“这个常量的引用，所以它们指向的地址是相同的，“==”判断就是为true了，而str3是新建了一个对象，内存中重新开辟了存储空间，此时与之前“String”常量的存储位置是不同的，所以当“==”判断时，就是false了。

当调用.equals();方法，这个方法是Object类提供的，由子类去重写，其中在Object类中，.equals();和“==”是相等的，Object类中的方法：

```java
public boolean equals(Object obj) { 
     return (this == obj); 
 }
```

只是在String类中重写了该方法，String类中的方法：

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

当比较后的引用不相等时，它就会比较值是否相等。