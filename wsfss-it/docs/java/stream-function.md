---
comments: true
---

## Stream & Function

不少场景下，需要将集合中的元素映射到另一个集合中，或者将集合中的元素进行分组等，这种场景下，可以使用 Stream API。同时，结合函数式编程思想，可以将这些操作抽象成一个函数，然后通过函数调用的方式来完成。这里提供了一些常用组合的方法。

### Collection to Stream

我们想要将集合中指定字段组成一个新的数据流，并结合函数式编程抽象出一个公共方法。

=== "Java"

```java linenums="1"
/**
 * @description: 将集合中指定字段组成一个新的流
 * @param coll 待转换的集合
 * @param feild 需要转换的字段
 */
private static <T, R> Stream<R> listFeildStream(Collection<T> coll, Function<? super T, ? extends R> feild) {
    BiFunction<Collection<T>, Function<? super T, ? extends R>, Stream<R>> func = (t, u) -> t.stream().map(u);
    return func.apply(coll, feild);
}
```

### List to Map

将List集合转换为Map集合，我们定义了两个入参，第一个参数是待转换的List集合，第二个参数是方法返回结果中Map集合中元素的 `key` 值。这里的入参 `key` 是一个函数，该函数返回List集合中元素的某个属性值。

方法返回参数是Map集合，Map集合的key是入参key函数返回值，Map集合的value是List集合中的对象。

=== "Java"

```java linenums="1"
/**
 * @description: 将List集合转换为Map集合
 * @param list 待转换的List集合
 * @param key Map集合的key
 */
public static <T, R> Map<R, T> listToMapIdentity(List<T> list, Function<? super T, ? extends R> key) {
    BiFunction<List<T>, Function<? super T, ? extends R>, Map<R, T>> func = (t, u) -> t.stream().collect(Collectors.toMap(u, Function.identity(), (v1,v2) -> v2));
    return func.apply(list, key);
}
```

有时候我们并不需要映射到List集合中的对象，而是需要映射到List集合中的对象属性，可以稍微改造一下上面的方法。我们增加了一个参数 `value` ，用于映射到List集合中的对象属性。

=== "Java"

```java linenums="1"
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
```