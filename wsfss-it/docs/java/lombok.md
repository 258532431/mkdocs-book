---
comments: true
---

## 安装 Install

要安装 `Lombok` 依赖，添加下面的内容到你的`pom.xml` 文件中的 `<dependencies>`区块：

```xml
<dependencies>
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.34</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

Jetbrains IntelliJ IDEA编辑器在2020.3到2023.1版本之间不需要插件就可以与lombok兼容。对于2020.3之前或2023.1之后的版本，您可以添加IntelliJ插件来添加对IntelliJ的支持:

- 选择 <font color="red">File > Settings > Plugins</font>
- 点击 <font color="red">Browse repositories...</font>
- 搜索 <font color="red">Lombok Plugin</font>
- 点击 <font color="red">Install plugin</font>
- 重启 IntelliJ IDEA

## 特性 Features

javadoc 中的 `Lombok` 文档是可用的，但官方建议我们依据官网文档说明使用。

### val

`val` 在lombok 0.10中引入，用于定义最终变量，相当于 `final`。但是这个特性只对局部变量和 foreach 循环起作用，对字段不起作用。

!!! note "警告"
    此功能目前在 NetBeans 中不起作用。

下面是官方给出的使用示例：

=== "用Lombok实现"

    ```java
    import java.util.ArrayList;
    import java.util.HashMap;
    import lombok.val;

    public class ValExample {
        public String example() {
            val example = new ArrayList<String>();
            example.add("Hello, World!");
            val foo = example.get(0);
            return foo.toLowerCase();
        }
        
        public void example2() {
            val map = new HashMap<Integer, String>();
            map.put(0, "zero");
            map.put(5, "five");
            for (val entry : map.entrySet()) {
                System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
            }
        }
    }
    ```

=== "对应Java实现"

    ```java
    import java.util.ArrayList;
    import java.util.HashMap;
    import java.util.Map;

    public class ValExample {
        public String example() {
            final ArrayList<String> example = new ArrayList<String>();
            example.add("Hello, World!");
            final String foo = example.get(0);
            return foo.toLowerCase();
        }
        
        public void example2() {
            final HashMap<Integer, String> map = new HashMap<Integer, String>();
            map.put(0, "zero");
            map.put(5, "five");
            for (final Map.Entry<Integer, String> entry : map.entrySet()) {
                System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
            }
        }
    }
    ```

!!! note "警告"
    在不明确的情况下，例如当初始化表达式为 `null` 时，其推断出的类型将为 `java.lang.Object`。

### var

`var` 在lombok 1.16.12中引入，其作用和 `val` 一样，只是变量没有被标记为 `final`。

### @NonNull

`@NonNull` 在lombok v0.11.10中引入，可以在方法参数或构造函数的参数上使用，lombok 会自动生成一个 `null` 检查语句。这个自动生成的 `null` 检查语句相当于 `if (param == null) throw new NullPointerException("param is marked non-null but is null");` ，并且该语句会被插入到方法的最顶端。对于构造函数， `null` 检查将立即插入任何显式 `this()` 或 `super()` 调用之后。

=== "用Lombok实现"

    ```java
    import lombok.NonNull;

    public class NonNullExample extends Something {
        private String name;
        
        public NonNullExample(@NonNull Person person) {
            super("Hello");
            this.name = person.getName();
        }
    }
    ```

=== "对应Java实现"

    ```java
    import lombok.NonNull;

    public class NonNullExample extends Something {
        private String name;
        
        public NonNullExample(@NonNull Person person) {
            super("Hello");
            if (person == null) {
                throw new NullPointerException("person is marked non-null but is null");
            }
            this.name = person.getName();
        }
    }
    ```

### @Cleanup

自动资源管理，安全地调用close()方法。

您可以使用 `@Cleanup` 来确保在代码执行路径退出当前作用域之前自动清理给定的资源。例如。这样用 `@Cleanup` 注释任何局部变量声明: `@Cleanup InputStream in = new FileInputStream("some/file");`，这将导致当前作用域结束时，会调用 `in.close()`。这个调用保证通过 `try/finally` 构造运行。

如果你想要清理的对象类型没有 `close()` 方法，而是其他无参数方法，你可以像这样指定这个方法的名称: `@Cleanup("dispose") org.eclipse.swt.widgets.CoolBar bar = new CoolBar(parent, 0);`，不指定方法名的情况下，清理方法默认为 `close()`，且带有1个或多个参数的清理方法不能通过`@Cleanup` 调用。

=== "用Lombok实现"

    ```java
    import lombok.Cleanup;
    import java.io.*;

    public class CleanupExample {
        public static void main(String[] args) throws IOException {
            @Cleanup InputStream in = new FileInputStream(args[0]);
            @Cleanup OutputStream out = new FileOutputStream(args[1]);
            byte[] b = new byte[10000];
            while (true) {
                int r = in.read(b);
                if (r == -1) break;
                out.write(b, 0, r);
            }
        }
    }
    ```

=== "对应Java实现"

    ```java
    import java.io.*;

    public class CleanupExample {
        public static void main(String[] args) throws IOException {
            InputStream in = new FileInputStream(args[0]);
            try {
                OutputStream out = new FileOutputStream(args[1]);
                try {
                    byte[] b = new byte[10000];
                    while (true) {
                        int r = in.read(b);
                        if (r == -1) break;
                        out.write(b, 0, r);
                    }
                } finally {
                    if (out != null) {
                        out.close();
                    }
                }
            } finally {
                if (in != null) {
                    in.close();
                }
            }
        }
    }
    ```

!!! note "警告"
    如果您的代码抛出异常，并且随后触发的清理方法调用也抛出异常，那么原始异常将被清理调用抛出的异常所覆盖。

### @Getter and @Setter

您可以用 `@Getter` 或 `@Setter` 注释任何字段，以让 `lombok` 自动生成默认的getter/setter方法。默认getter只是返回字段，如果字段被称为foo，则命名为getFoo(如果字段的类型是布尔型，则命名为isFoo)。如果字段名为foo，则默认setter被命名为setFoo，返回void，并接受一个与该字段相同类型的参数。它只是将字段设置为这个值。如果在类上添加该注释，则会为那个类中的所有非静态字段生成getter/setter 方法。

生成的getter/setter方法是 `public` 的，除非您显式指定 AccessLevel，如下面的示例所示。合法访问级别包括 PUBLIC、PROTECTED、PACKAGE 和PRIVATE。

lombok v1.12.0中的新功能: 字段上的 `javadoc` 将被复制到生成的getter和setter中。

=== "用Lombok实现"

    ```java
    import lombok.AccessLevel;
    import lombok.Getter;
    import lombok.Setter;

    public class GetterSetterExample {
        /**
         * Age of the person. Water is wet.
         * 
         * @param age New value for this person's age. Sky is blue.
         * @return The current value of this person's age. Circles are round.
         */
        @Getter @Setter private int age = 10;
        
        /**
         * Name of the person.
         * -- SETTER --
         * Changes the name of this person.
         * 
         * @param name The new value.
         */
        @Setter(AccessLevel.PROTECTED) private String name;
        
        @Override public String toString() {
            return String.format("%s (age: %d)", name, age);
        }
    }
    ```

=== "对应Java实现"

    ```java
    public class GetterSetterExample {
        /**
         * Age of the person. Water is wet.
         */
        private int age = 10;

        /**
         * Name of the person.
         */
        private String name;
        
        @Override public String toString() {
            return String.format("%s (age: %d)", name, age);
        }
        
        /**
         * Age of the person. Water is wet.
         *
         * @return The current value of this person's age. Circles are round.
         */
        public int getAge() {
            return age;
        }
        
        /**
         * Age of the person. Water is wet.
         *
         * @param age New value for this person's age. Sky is blue.
         */
        public void setAge(int age) {
            this.age = age;
        }
        
        /**
         * Changes the name of this person.
         *
         * @param name The new value.
         */
        protected void setName(String name) {
            this.name = name;
        }
    }
    ```

如果已经存在任何具有相同名称(不区分大小写)和相同参数的方法，则不会生成任何方法。

### @ToString

用 `@ToString` 注释类将自动生成 `toString()` 方法，生成格式是这样的：`MyClass(foo=123, bar=234)`。默认情况下将打印所有非静态字段，如果想跳过某些字段，可以使用 `@ToString.Exclude` 对这些字段进行注释。或者，您可以使用 `@ToString(onlyExplicitlyIncluded = true)` 指定您希望使用的字段，然后用 `@ToString.include` 标记您想要包含的每个字段。通过将 `includeFieldNames` 参数设置为 `true`，可以输出字段名称。

通过将 `callSuper` 设置为true，您可以将 `toString` 的父类实现的输出包含到输出中。您还可以在 `toString` 中包含方法调用的输出，只能包含不带参数的非静态方法，用 `@ToString.Include` 标记该方法即可。

=== "用Lombok实现"

    ```java
    import lombok.ToString;

    @ToString
    public class ToStringExample {
        private static final int STATIC_VAR = 10;
        private String name;
        private Shape shape = new Square(5, 10);
        private String[] tags;
        @ToString.Exclude private int id;
        
        public String getName() {
            return this.name;
        }
        
        @ToString(callSuper=true, includeFieldNames=true)
        public static class Square extends Shape {
            private final int width, height;
            
            public Square(int width, int height) {
                this.width = width;
                this.height = height;
            }
        }
    }
    ```

=== "对应Java实现"

    ```java
    import java.util.Arrays;

    public class ToStringExample {
        private static final int STATIC_VAR = 10;
        private String name;
        private Shape shape = new Square(5, 10);
        private String[] tags;
        private int id;
        
        public String getName() {
            return this.name;
        }
        
        public static class Square extends Shape {
            private final int width, height;
            
            public Square(int width, int height) {
                this.width = width;
                this.height = height;
            }
            
            @Override public String toString() {
                return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
            }
        }
        
        @Override public String toString() {
            return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
        }
    }
    ```

### @EqualsAndHashCode

根据对象的字段自动生成 `hashCode` 和 `equals` 实现。

### @NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor

作用于类，用于生成无参构造函数、有参构造函数和全参构造函数。

`@NoArgsConstructor` 将自动生成一个无参构造函数。

`@RequiredArgsConstructor` 将自动生成一个包含所有必需字段的构造函数。

`@AllArgsConstructor` 将自动生成一个包含所有字段的构造函数。

### @Data

`@Data` 是一个方便的快捷注释，它将 `@ToString、@EqualsAndHashCode、@Getter / @Setter` 和 `@RequiredArgsConstructor` 的 特性捆绑在一起，换句话说，`@Data` 相当于这些注解的组合，并且这些字段已标记为 `@NonNull`，以确保字段永远不会为空。

=== "用Lombok实现"

    ```java
    import lombok.AccessLevel;
    import lombok.Setter;
    import lombok.Data;
    import lombok.ToString;

    @Data public class DataExample {
        private final String name;
        @Setter(AccessLevel.PACKAGE) private int age;
        private double score;
        private String[] tags;
        
        @ToString(includeFieldNames=true)
        @Data(staticConstructor="of")
        public static class Exercise<T> {
            private final String name;
            private final T value;
        }
    }
    ```

=== "对应Java实现"

    ```java
    import java.util.Arrays;

    public class DataExample {
        private final String name;
        private int age;
        private double score;
        private String[] tags;
        
        public DataExample(String name) {
            this.name = name;
        }
        
        public String getName() {
            return this.name;
        }
        
        void setAge(int age) {
            this.age = age;
        }
        
        public int getAge() {
            return this.age;
        }
        
        public void setScore(double score) {
            this.score = score;
        }
        
        public double getScore() {
            return this.score;
        }
        
        public String[] getTags() {
            return this.tags;
        }
        
        public void setTags(String[] tags) {
            this.tags = tags;
        }
        
        @Override public String toString() {
            return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
        }
        
        protected boolean canEqual(Object other) {
            return other instanceof DataExample;
        }
        
        @Override public boolean equals(Object o) {
            if (o == this) return true;
            if (!(o instanceof DataExample)) return false;
            DataExample other = (DataExample) o;
            if (!other.canEqual((Object)this)) return false;
            if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
            if (this.getAge() != other.getAge()) return false;
            if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
            if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
            return true;
        }
        
        @Override public int hashCode() {
            final int PRIME = 59;
            int result = 1;
            final long temp1 = Double.doubleToLongBits(this.getScore());
            result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
            result = (result*PRIME) + this.getAge();
            result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
            result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
            return result;
        }
        
        public static class Exercise<T> {
            private final String name;
            private final T value;
            
            private Exercise(String name, T value) {
                this.name = name;
                this.value = value;
            }
            
            public static <T> Exercise<T> of(String name, T value) {
                return new Exercise<T>(name, value);
            }
            
            public String getName() {
                return this.name;
            }
            
            public T getValue() {
                return this.value;
            }
            
            @Override public String toString() {
                return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
            }
            
            protected boolean canEqual(Object other) {
                return other instanceof Exercise;
            }
            
            @Override public boolean equals(Object o) {
                if (o == this) return true;
                if (!(o instanceof Exercise)) return false;
                Exercise<?> other = (Exercise<?>) o;
                if (!other.canEqual((Object)this)) return false;
                if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
                if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
                return true;
            }
            
            @Override public int hashCode() {
                final int PRIME = 59;
                int result = 1;
                result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
                result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
                return result;
            }
        }
    }
    ```

### @Value

### @Builder

### @SneakyThrows

### @Synchronized

### @Locked

### @With

### @Getter(lazy=true)

### @Log

### experimental

#### @Accessors

#### @Delegate

#### @StandardException