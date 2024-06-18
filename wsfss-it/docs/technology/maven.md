## 常用类库

很多知名大厂都提供了丰富的工具包类库，以下列举了一些常用的类库。

=== "pom.xml"

    ```xml
    <dependencies>

        <!-- @Data 自动get、set -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.8</version>
        </dependency>
        
        <!-- 工具类大集合 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>
        
        <!-- 字符串工具类 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.8.1</version>
        </dependency>
        
        <!-- alibaba的json工具类 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.73</version>
        </dependency>

        <!-- google工具类 -->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>31.1-jre</version>
        </dependency>
        
    </dependencies>
    ```

:arrow_up: [<font size="2">回到顶部</font>][top]

[top]: #