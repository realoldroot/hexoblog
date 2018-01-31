---
title: commons-lang3的tuple包介绍
date: 2018-01-31 19:10:39
tags: java
---


### 问题
有时候我们调用方法的时候返回值有可能不止1个，比如说我们返回`name = "张三" age = 20`这两个值，那么这个时候方法的返回值怎么写呢。在Scala和python中都有Tuple使用，在java中我们怎么办。很容易我们就想到了一个方式：可以构造一个User类来封装这两个属性。
但是如果返回的两个值并没有任何关联关系，或者说每一个方法返回的参数都不同，那么我们就得为每一个方法的返回类型去创建对应的类来取包装，或许还有其他的解决方式，比如说返回一个map，返回一个List，返回一个array都可以。
使用map作为返回值的话调用方在不清楚map中具体有什么内容的时候需要去遍历keySet或entrySet,而list和array也是同样的问题，不知道哪一个参数存放在哪里。

<!-- more -->


### 解决方式
其实前人已经为我们解决了这个问题。
如果你经常使用apache的commons-lang3包来处理String的null判断，那么你依旧可以从这个包中找到我们想要的一个所谓的**元组**。
`org.apache.commons.lang3.tuple.Pair`

```java 

/**
 * <p>A pair consisting of two elements.</p>
 *
 * <p>This class is an abstract implementation defining the basic API.
 * It refers to the elements as 'left' and 'right'. It also implements the
 * {@code Map.Entry} interface where the key is 'left' and the value is 'right'.</p>
 *
 * <p>Subclass implementations may be mutable or immutable.
 * However, there is no restriction on the type of the stored objects that may be stored.
 * If mutable objects are stored in the pair, then the pair itself effectively becomes mutable.</p>
 *
 * @param <L> the left element type
 * @param <R> the right element type
 *
 * @since Lang 3.0
 */
public abstract class Pair<L, R> implements Map.Entry<L, R>, Comparable<Pair<L, R>>, Serializable {

    /** Serialization version */
    private static final long serialVersionUID = 4954918890077093841L;

    /**
     * <p>Obtains an immutable pair of from two objects inferring the generic types.</p>
     *
     * <p>This factory allows the pair to be created using inference to
     * obtain the generic types.</p>
     *
     * @param <L> the left element type
     * @param <R> the right element type
     * @param left  the left element, may be null
     * @param right  the right element, may be null
     * @return a pair formed from the two parameters, not null
     */
    public static <L, R> Pair<L, R> of(final L left, final R right) {
        return new ImmutablePair<>(left, right);
    }

    //-----------------------------------------------------------------------
    /**
     * <p>Gets the left element from this pair.</p>
     *
     * <p>When treated as a key-value pair, this is the key.</p>
     *
     * @return the left element, may be null
     */
    public abstract L getLeft();

    /**
     * <p>Gets the right element from this pair.</p>
     *
     * <p>When treated as a key-value pair, this is the value.</p>
     *
     * @return the right element, may be null
     */
    public abstract R getRight();

    /**
     * <p>Gets the key from this pair.</p>
     *
     * <p>This method implements the {@code Map.Entry} interface returning the
     * left element as the key.</p>
     *
     * @return the left element as the key, may be null
     */
    @Override
    public final L getKey() {
        return getLeft();
    }

    /**
     * <p>Gets the value from this pair.</p>
     *
     * <p>This method implements the {@code Map.Entry} interface returning the
     * right element as the value.</p>
     *
     * @return the right element as the value, may be null
     */
    @Override
    public R getValue() {
        return getRight();
    }

    //-----------------------------------------------------------------------
    /**
     * <p>Compares the pair based on the left element followed by the right element.
     * The types must be {@code Comparable}.</p>
     *
     * @param other  the other pair, not null
     * @return negative if this is less, zero if equal, positive if greater
     */
    @Override
    public int compareTo(final Pair<L, R> other) {
      return new CompareToBuilder().append(getLeft(), other.getLeft())
              .append(getRight(), other.getRight()).toComparison();
    }

    /**
     * <p>Compares this pair to another based on the two elements.</p>
     *
     * @param obj  the object to compare to, null returns false
     * @return true if the elements of the pair are equal
     */
    @Override
    public boolean equals(final Object obj) {
        if (obj == this) {
            return true;
        }
        if (obj instanceof Map.Entry<?, ?>) {
            final Map.Entry<?, ?> other = (Map.Entry<?, ?>) obj;
            return Objects.equals(getKey(), other.getKey())
                    && Objects.equals(getValue(), other.getValue());
        }
        return false;
    }

    /**
     * <p>Returns a suitable hash code.
     * The hash code follows the definition in {@code Map.Entry}.</p>
     *
     * @return the hash code
     */
    @Override
    public int hashCode() {
        // see Map.Entry API specification
        return (getKey() == null ? 0 : getKey().hashCode()) ^
                (getValue() == null ? 0 : getValue().hashCode());
    }

    /**
     * <p>Returns a String representation of this pair using the format {@code ($left,$right)}.</p>
     *
     * @return a string describing this object, not null
     */
    @Override
    public String toString() {
        return new StringBuilder().append('(').append(getLeft()).append(',').append(getRight()).append(')').toString();
    }

    /**
     * <p>Formats the receiver using the given format.</p>
     *
     * <p>This uses {@link java.util.Formattable} to perform the formatting. Two variables may
     * be used to embed the left and right elements. Use {@code %1$s} for the left
     * element (key) and {@code %2$s} for the right element (value).
     * The default format used by {@code toString()} is {@code (%1$s,%2$s)}.</p>
     *
     * @param format  the format string, optionally containing {@code %1$s} and {@code %2$s}, not null
     * @return the formatted string, not null
     */
    public String toString(final String format) {
        return String.format(format, getLeft(), getRight());
    }

}
```

这个类构建了一个由两个元素组成的一个**元组**，`left`和`right`两个元素。
继承了Map.Entry，可以使用`getKey()`和`getValue()`方法，从源码中我们也发现其实是调用了他本身的`getLeft()`和`getRight()`方法，而这两个方法是需要子类去实现，我们后面说。
继承了Comparable，可以比较两个Pair。
继承了Serializable，可以被序列化。
比较重要的就是`getKey()`和`getValue()`，下面我们看这个类的两个实现`ImmutablePair`和`MutablePair`。

`ImmutablePair`很好理解，不可变的Pair，相对的`MutablePair`就是可变的Pair

先看`ImmutablePair.class`
```java

/**
 * <p>An immutable pair consisting of two {@code Object} elements.</p>
 *
 * <p>Although the implementation is immutable, there is no restriction on the objects
 * that may be stored. If mutable objects are stored in the pair, then the pair
 * itself effectively becomes mutable. The class is also {@code final}, so a subclass
 * can not add undesirable behaviour.</p>
 *
 * <p>#ThreadSafe# if both paired objects are thread-safe</p>
 *
 * @param <L> the left element type
 * @param <R> the right element type
 *
 * @since Lang 3.0
 */
public final class ImmutablePair<L, R> extends Pair<L, R> {

    /**
     * An immutable pair of nulls.
     */
    // This is not defined with generics to avoid warnings in call sites.
    @SuppressWarnings("rawtypes")
    private static final ImmutablePair NULL = ImmutablePair.of(null, null);

    /** Serialization version */
    private static final long serialVersionUID = 4954918890077093841L;

    /**
     * Returns an immutable pair of nulls.
     *
     * @param <L> the left element of this pair. Value is {@code null}.
     * @param <R> the right element of this pair. Value is {@code null}.
     * @return an immutable pair of nulls.
     * @since 3.6
     */
    @SuppressWarnings("unchecked")
    public static <L, R> ImmutablePair<L, R> nullPair() {
        return NULL;
    }

    /** Left object */
    public final L left;
    /** Right object */
    public final R right;

    /**
     * <p>Obtains an immutable pair of from two objects inferring the generic types.</p>
     *
     * <p>This factory allows the pair to be created using inference to
     * obtain the generic types.</p>
     *
     * @param <L> the left element type
     * @param <R> the right element type
     * @param left  the left element, may be null
     * @param right  the right element, may be null
     * @return a pair formed from the two parameters, not null
     */
    public static <L, R> ImmutablePair<L, R> of(final L left, final R right) {
        return new ImmutablePair<>(left, right);
    }

    /**
     * Create a new pair instance.
     *
     * @param left  the left value, may be null
     * @param right  the right value, may be null
     */
    public ImmutablePair(final L left, final R right) {
        super();
        this.left = left;
        this.right = right;
    }

    //-----------------------------------------------------------------------
    /**
     * {@inheritDoc}
     */
    @Override
    public L getLeft() {
        return left;
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public R getRight() {
        return right;
    }

    /**
     * <p>Throws {@code UnsupportedOperationException}.</p>
     *
     * <p>This pair is immutable, so this operation is not supported.</p>
     *
     * @param value  the value to set
     * @return never
     * @throws UnsupportedOperationException as this operation is not supported
     */
    @Override
    public R setValue(final R value) {
        throw new UnsupportedOperationException();
    }

}
```

`public final L left;`和`public final R right;`两个不可变的值注定了这个类是不可变的。
在构造方法`ImmutablePair(final L left, final R right)`中初始化这两个值。
本类实现了 `getLeft()`和`getRight()`方法，返回`left`和`right`。
当我们试图调用`setValue()`方法的时候会抛出`UnsupportedOperationException`异常，因为这一对值是不可变的。

`ImmutablePair.class`
```java
 public void setRight(final R right) {
        this.right = right;
    }
public void setLeft(final L left) {
    this.left = left;
}
public R setValue(final R value) {
        final R result = getRight();
        setRight(value);
        return result;
    }
```

`MutablePair`和`ImmutablePair`类基本一样，只是在定义`public L left;`和`public R right;`没有使用`final`类型的。注定了可以修改这两个值。并且提供了setter方法可以供修改这一对值，并且提供了`setValuef()`方法修改`right`值。

现在我们可以愉快的使用这个类来作为返回两个参数的方法的返回值了。

### 提问 ###
如果你想问，我返回的参数不是2个，是3个怎么办？

没问题，一样满足你，在这个`org.apache.commons.lang3.tuple`包中提供了针对构建三个元素的`Triple`类，类定义中`abstract class Triple<L, M, R>`。定义了3个泛型同样提供了`ImmutableTriple`和`MutableTriple`一对不可变和可变的实现类。

你过你还问我，我返回的参数是4个怎么办，还有什么类可以供我使用吗？

抱歉，或许你需要定义java bean 去实现你的复杂返回值：）。