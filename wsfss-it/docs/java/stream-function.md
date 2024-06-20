---
comments: true
---

## Stream & Function

不少场景下，需要将集合中的元素映射到另一个集合中，或者将集合中的元素进行分组等，这种场景下，可以使用 Stream API。同时，结合函数式编程思想，可以将这些操作抽象成一个函数，然后通过函数调用的方式来完成。这里提供了一些常用组合的方法。

我们创建一个对象，用于测试。

=== "User.java"

    ```java linenums="1"
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class User {

        private Long id;
        private String name;
        private Integer age;

    }
    ```

### Collection to Stream

我们想要将集合中指定字段组成一个新的数据流，并结合函数式编程抽象出一个公共方法。

=== "Util.java"

    ```java linenums="1"
    public class Util {

        /**
         * @description: 将集合中指定字段组成一个新的流
         * @param coll 待转换的集合
         * @param feild 需要转换的字段
         */
        private static <T, R> Stream<R> listFeildStream(Collection<T> coll, Function<? super T, ? extends R> feild) {
            BiFunction<Collection<T>, Function<? super T, ? extends R>, Stream<R>> func = (t, u) -> t.stream().map(u);
            return func.apply(coll, feild);
        }
        
    }
    ```

=== "Test.java"

    ```java linenums="1"
    public class Test {

        public static void main(String[] args) {

            List<User> userList = new ArrayList<>();
            userList.add(new User("张三", 18, "北京"));
            userList.add(new User("李四", 20, "上海"));

            Stream<String> stream = Util.listFeildStream(userList, User::getName);

        }
        
    }
    ```

### List to Map

将List集合转换为Map集合，我们定义了两个入参，第一个参数是待转换的List集合，第二个参数是方法返回结果中Map集合中元素的 `key` 值。这里的入参 `key` 是一个函数，该函数返回List集合中元素的某个属性值。

方法返回参数是Map集合，Map集合的key是入参key函数返回值，Map集合的value是List集合中的对象。

!!! info "提示"

    这里使用了函数 `(v1,v2) -> v2` 防止集合转换中 key 重复造成的异常。

=== "Util.java"

    ```java linenums="1"
    public class Util {

        /**
         * @description: 将List集合转换为Map集合
         * @param list 待转换的List集合
         * @param key Map集合的key
         */
        public static <T, R> Map<R, T> listToMapIdentity(List<T> list, Function<? super T, ? extends R> key) {
            BiFunction<List<T>, Function<? super T, ? extends R>, Map<R, T>> func = (t, u) -> t.stream().collect(Collectors.toMap(u, Function.identity(), (v1,v2) -> v2));
            return func.apply(list, key);
        }

    }
    ```

=== "Test.java"

    ```java linenums="1"
    public class Test {

        public static void main(String[] args) {

            List<User> userList = new ArrayList<>();
            userList.add(new User("张三", 18, "北京"));
            userList.add(new User("李四", 20, "上海"));

            Map<Long, User> map = Util.listToMapIdentity(userList, User::getId);

        }
        
    }
    ```

有时候我们并不需要映射到List集合中的对象，而是需要映射到List集合中的对象属性，可以稍微改造一下上面的方法。我们增加了一个参数 `value` ，用于映射到List集合中的对象属性。

=== "Util.java"

    ```java linenums="1"
    public class Util {

        /**
         * @description: 将List集合转换为Map集合
         * @param list 待转换的List集合
         * @param key Map集合的key
         * @param value Map集合的value
         */
        public static <T, K, V> Map<K, V> listToMap(List<T> list, Function<? super T, ? extends K> key, Function<? super T, ? extends V> value) {
            Pair<Function<? super T, ? extends K>, Function<? super T, ? extends V>> pair = Pair.of(key, value);
            BiFunction<List<T>, Pair<Function<? super T, ? extends K>, Function<? super T, ? extends V>>, Map<K, V>> f2 = (t, u) -> t.stream().collect(Collectors.toMap(u.getLeft(), u.getRight(), (v1,v2) -> v2));
            return f2.apply(list, pair);
        }

    }
    ```

=== "Test.java"

    ```java linenums="1"
    public class Test {

        public static void main(String[] args) {

            List<User> userList = new ArrayList<>();
            userList.add(new User("张三", 18, "北京"));
            userList.add(new User("李四", 20, "上海"));

            Map<Long, String> map = Util.listToMap(userList, User::getId, User::getName);

        }
        
    }
    ```

假设我们有一个List集合，其中包含多个对象，每个对象都有一个 `age` 属性，我们希望根据这个属性进行分组。

=== "Util.java"

    ```java linenums="1"
    public class Util {

        /**
         * @description: 将List集合按指定对象的属性分组
         * @param list 待转换的List集合
         * @param feild 分组依据的字段
         */
        public static <T, R> Map<R, List<T>> groupFeild(List<T> list, Function<? super T, ? extends R> feild) {
            BiFunction<List<T>, Function<? super T, ? extends R>, Map<R, List<T>>> func = (t, u) -> t.stream().collect(Collectors.groupingBy(f));
            return func.apply(list, feild);
        }

    }
    ```

=== "Test.java"

    ```java linenums="1"
    public class Test {

        public static void main(String[] args) {

            List<User> userList = new ArrayList<>();
            userList.add(new User("张三", 18, "北京"));
            userList.add(new User("李四", 20, "上海"));

            Map<Integer, List<User>> map = Util.groupFeild(userList, User::getAge);

        }
        
    }
    ```