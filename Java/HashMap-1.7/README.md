
HashMap是用于映射(键值对)处理的数据类型：它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。 HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

## Java 1.7 HashMap源码
读完源码 所了解的知识：
- 1.HashMap属性变量
- 2.构造函数
- 3.怎么计算出指定容量最近的二次幂（roundUpToPowerOf2）
- 4.如何计算出key的hash所在数组位置
- 4.存放与获取键值对（get 与 put）
- 5.小了解Holder静态类，如何人工干预hash，减小hash碰撞


```
/*
 * Copyright (c) 1997, 2010, Oracle and/or its affiliates. All rights reserved.
 * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
 *
 */
import java.io.*;
import java.util.*;

/**
 *
 * 1.该类是基于哈希表实现的映射接口。该实现提供了所有可选的映射操作和许可
 * 空值和空键。(除了HashMap是不同步的并且允许空值之外，HashMap大致相当于Hashtable)
 * 该类不能保证映射的顺序;特别是，它不保证该顺序恒久不变。
 *
 * 解读：说明了HashMap是基于hash表来实现的，具备了map的特性功能。这里提到的
 * ‘该类不能保证映射的顺序’ 因为hashMap是根据key的hashCode方法计算hash值，从而得到具体hash桶的位置，所以具备顺序具有不确定性。
 * ‘它不保证该顺序恒久不变’ 是因为HashMap在初始化的容量是一定的，当插入的元素多起来的时候，元素很容易发生hash值碰撞，
 * 每次碰撞都会遍历链表，导致插入和读取效率低下。而这个时候，HashMap需要自己扩容，所以需要重新计算所有元素的Hash值，重新分配hash桶。
 *
 *
 *
 *
 * 2.该实现 在假设哈希函数将元素适当地分散在桶中的前提下，为基本操作(get和put)提供了稳定的性能。
 * 对集合视图的迭代所需的时间与HashMap实例的“容量”(桶的数量)加上它的大小(键-值映射关系的数量)成正比。
 * 因此，如果迭代性能很重要，那么不要将初始容量设置得太高(或负载系数设置得太低)，这一点非常重要。
 *
 *
 *
 * 3.HashMap的实例有两个影响其性能的参数:初始容量和装载因子。
 * 容量是哈希表中的桶数，初始容量就是创建哈希表时的容量。
 * 装载因子是一种度量方法，用来衡量在自动增加哈希表的容量之前，哈希表允许的最大长度。
 * 当哈希表中的条目数超过负载因子和当前容量的乘积时，哈希表将被重新哈希(即重新构建内部数据结构)，这样哈希表的桶数大约是原来的两倍。
 *
 * 解读：说明了HashMap有两个影响性能的关键参数：capacity 和 loadFactor。
 * 那为什么这么说呢，因为底层进行hash进行存放的位置 初始容量会有关联，可以人工进行干预---initHashSeedAsNeeded方法
 * 而负载因子与初始容量 决定了阈值，即扩容的时机（ 负载因子x初始容量 >= threshold），因为扩容会需要重新创建大的新数组， 并且把原先的数据复制到新数组会有性能损失。
 *
 *
 *
 *
 * 4.通常，默认加载因子 (.75) 在时间和空间成本上寻求一种折衷。
 * 加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，
 * 包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，
 * 以便最大限度地降低 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。
 *
 *
 *
 * 5.如果很多映射关系要存储在 HashMap 实例中，则相对于按需执行自动的 rehash 操作以增大表的容量来说，
 * 使用足够大的初始容量创建它将使得映射关系能更有效地存储。
 *
 * 解读：适当的提高所设置的初始容量，也就是空间换时间 来更高效地进行存储。
 *

 *
 * 6.注意，此实现不是同步的。如果多个线程同时访问此映射，而其中至少一个线程从结构上修改了该映射，
 * 则它必须 保持外部同步。（结构上的修改是指添加或删除一个或多个映射关系的操作；
 * 仅改变与实例已经包含的键关联的值不是结构上的修改。）
 * 这一般通过对自然封装该映射的对象进行同步操作来完成
 *
 *
 *
 *
 * 7.如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。
 * 最好在创建时完成这一操作，以防止对映射进行意外的不同步访问，如下所示：
 *   Map m = Collections.synchronizedMap(new HashMap(...));
 *
 *

 *
 * 8.由所有此类的“集合视图方法”所返回的迭代器都是快速失败的：
 *  在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器自身的 remove 或 add 方法，
 *  其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。
 *  因此，面对并发的修改，迭代器很快就会完全快速失败，而不冒在将来不确定的时间任意发生不确定行为的风险。
 *
 * 解读：对集合遍历的同时又进行操作 HashMap会快速失败（modCount：修改次数，靠此参数实现），
 *      但是通过集合的迭代器进行添加删除是允许的
 *
 *
 * 9.注意，迭代器的快速失败行为不能得到保证，一般来说，存在不同步的并发修改时，
 * 不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。
 * 因此，编写依赖于此异常程序的方式是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。
 *
 *  带数字编号都是文档翻译
 */

public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
{

    /**
     * 默认初始容量-必须是2的幂次方。
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 最大容量,
     * 如果任何一个带参数的构造函数隐式指定了较高的值，则使用MAXIMUM_CAPACITY。
     * 必须是2的幂次方， 同时也要小于 1<<30.
     *
     * 总的意思：如果实例一个HashMap对象指定了初始容量 大于了1<<30，则直接等于1<<30。
     * 参考：public HashMap(int initialCapacity, float loadFactor)方法
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 在构造函数中未指定时使用的负载因子。当表容量达到3/4时才会再散列）
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 当表没有扩容时，要共享的空表实例。
     */
    static final Entry<?,?>[] EMPTY_TABLE = {};

    /**
     * 根据需要调整表的大小。长度必须是2的幂。
     */
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

    /**
     * 此映射中包含的键-值映射的数目。
     */
    transient int size;

    /**
     * 需要调整大小的阈值  (capacity * load factor).
     * 如果table == EMPTY_TABLE，那么这是扩容时创建表的初始容量。
     *
     * @serial
     */
    int threshold;

    /**
     * 哈希表的负载因子。
     *
     * @serial
     */
    final float loadFactor;

    /**
     *
     * 结构修改次数是指那些改变HashMap中映射数量或修改其内部结构的修改次数(例如，重新hash计算)。
     * 此字段用于使HashMap的集合视图上的迭代器快速失效。(见ConcurrentModificationException)。
     *
     */
    transient int modCount;

    /**
     * 映射容量的默认阈值，在该阈值之上，可选散列用于字符串键。可选哈希减少了由于字符串键的弱哈希码计算而产生的冲突。
     * 可以通过定义系统属性覆盖此值{@code jdk.map.althashing.threshold}.
     * 属性值{@code 1}强制在任何时候使用可选散列，而{@code -1}值确保永远不会使用可选散列。
     *
     * 总的来说：表示在对字符串键（即key为String类型）的HashMap应用替代哈希函数时HashMap的条目数量的默认阈值。
     *          替代哈希函数的使用可以减少由于对字符串键进行弱哈希码计算时的碰撞概率
     */
    static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;

    /**
     * 保存在VM启动后才能初始化的值。
     */
    private static class Holder {

        /**
         * 可切换到使用可选散列的阈值。主要参与减少hash碰撞使用。
         */
        static final int ALTERNATIVE_HASHING_THRESHOLD;

        static {
            //Java启动时带参 -Djdk.map.althashing.threshold
            String altThreshold = java.security.AccessController.doPrivileged(
                new sun.security.action.GetPropertyAction(
                    "jdk.map.althashing.threshold"));

            int threshold;
            try {
                //如果有传参 则使用该值，没有则使用默认最大值
                threshold = (null != altThreshold)
                        ? Integer.parseInt(altThreshold)
                        : ALTERNATIVE_HASHING_THRESHOLD_DEFAULT;

                // disable alternative hashing if -1
                //传入-1 则标识失效
                if (threshold == -1) {
                    threshold = Integer.MAX_VALUE;
                }
                //到达这步，不允许小于0
                if (threshold < 0) {
                    throw new IllegalArgumentException("value must be positive integer.");
                }
            } catch(IllegalArgumentException failed) {
                throw new Error("Illegal value for 'jdk.map.althashing.threshold'", failed);
            }
            //赋值可选hash阈值
            ALTERNATIVE_HASHING_THRESHOLD = threshold;
        }
    }

    /**
     * 与此实例相关联的随机值，应用于键的哈希码，使哈希冲突更难查找。如果为0，则禁用可选散列
     * 总的来说：该值会影响key的hash值，使它生成生成的hash更具随机性。 默认不起作用。
     */
    transient int hashSeed = 0;

    /**
     * 指定初始容量 跟 负载因子的构造函数，构造函数中判断了初始容量 跟 负载因子，不符合则进行抛出异常
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        //
        // 若初始容量为负, 抛出异常
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //若初始容量大于最大容量, 则初始容量等于最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //负载因子为负，或负载因子不是个数字将抛出异常
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();
    }

    /**
     * 指定初始容量 跟 使用默认负载因子(0.75)的构造函数
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 使用默认的初始容量(16)  跟 默认负载因子(0.75)的构造函数
     */
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 构造一个新的<tt>HashMap</tt>，使用与指定的<tt>Map</tt>相同的映射。
     * <tt>HashMap</tt>使用默认负载因子(0.75)和足以容纳指定<tt>Map</tt>中的映射的初始容量创建。
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        inflateTable(threshold);

        putAllForCreate(m);
    }

    private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        //如果number大于最大容量值 则等于最大容量值，否则取离大于等于number最近的2次方幂的值
        //这里需要一提的是Integer.highestOneBit(int)功能：给定一个数字，找到小于或等于这个数字的一个2的幂次方数。
        //               number - 1 是为了number本身就是2的幂次方
        return number >= MAXIMUM_CAPACITY
                ? MAXIMUM_CAPACITY
                : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
    }

    /**
     * inflateTable功能：对容量的2次幂调整，阈值的设定，数组的初始化以及hashSeed的设置
     */
    private void inflateTable(int toSize) {
        // Find a power of 2 >= toSize
        //对容量的2次幂调整  调至最接近该容量的2次幂大小
        int capacity = roundUpToPowerOf2(toSize);

        //防止超过最大容量，跟最大容量比较 取较小值，
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        //初始化数组
        table = new Entry[capacity];
        //hashSeed的设置 是否需要初始化hash种子
        initHashSeedAsNeeded(capacity);
    }

    // internal utilities

    /**
     * Initialization hook for subclasses. This method is called
     * in all constructors and pseudo-constructors (clone, readObject)
     * after HashMap has been initialized but before any entries have
     * been inserted.  (In the absence of this method, readObject would
     * require explicit knowledge of subclasses.)
     */
    void init() {
    }

    /**
     * 初始化哈希掩码值。我们推迟初始化，直到我们真正需要它。
     */
    final boolean initHashSeedAsNeeded(int capacity) {
        boolean currentAltHashing = hashSeed != 0;
        boolean useAltHashing = sun.misc.VM.isBooted() &&
                (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        boolean switching = currentAltHashing ^ useAltHashing;
        if (switching) {
            hashSeed = useAltHashing
                ? sun.misc.Hashing.randomHashSeed(this)
                : 0;
        }
        return switching;
    }

    /**
     * 检索对象哈希代码，并对结果哈希应用一个补充哈希函数，这可以防止质量较差的哈希函数。
     * 这一点非常关键，因为HashMap使用的是2次幂长度的哈希表，
     * 否则这些哈希表就会在较低位上没有差异的哈希码之间发生冲突。
     * 注意:空键总是映射到散列0，因此索引为0。
     *
     */
    final int hash(Object k) {
        int h = hashSeed;
        //如果hashSeed不等0 以及是字符串的键，则使用字符串hash方法
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }
        //，否则使用对象的hashCode方法与hashSeed异或 并进行二次扰动减少hash碰撞
        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        //   扰动的核心思想在于使计算出来的值在保留原有相关特性的基础上，增加其值的不确定性，从而降低冲突的概率。
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * 返回数组所在位置
     */
    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        //这里值得一提的是 h & (length-1) 等价于取余的功能，
        return h & (length-1);
    }

    /**
     * 返回此映射中键-值映射的数目。
     *
     * @return the number of key-value mappings in this map
     */
    public int size() {
        return size;
    }

    /**
     * 如果该映射不包含键-值映射，则返回<tt>true</tt>。
     *
     * @return <tt>true</tt> if this map contains no key-value mappings
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 返回指定键映射到的值，或{@code null}，如果此映射不包含键的映射。
     *
     * 更正式地说，如果这个映射包含一个从键{@code k}到值{@code v}的映射，那么{@code (key==null ?==null: key.equals(k))}，
     * 则该方法返回{@code v};否则返回{@code null}。(最多只能有一个这样的映射。)
     *
     * 返回值{@code null}不一定表示映射不包含键的映射;映射也可能显式地将键映射到{@code null}。
     * {@link #containsKey containsKey}操作可以用来区分这两种情况。
     *
     * @see #put(Object, Object)
     */
    public V get(Object key) {
        //1.如果key为空 则在table[0]中去找空key的值
        if (key == null)
            return getForNullKey();
        //2.不为空 则计算key的hash取余得到位置进行链表遍历比较得到entry
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

    /**
     * 卸载get()的版本，以查找空键。Null键映射到索引0。
     * 在两个最常用的操作(get和put)中，为了提高性能，将null情况拆分为单独的方法，但在其他操作中与条件语句合并。
     */
    private V getForNullKey() {
        //1.size为空,说明都没有存储数据 直接返回空
        if (size == 0) {
            return null;
        }
        //2.如果size不为空,由于key为null的元素，是放在table[0]这个链表中的。所以要找的话，直接到table[0]中查找就行了。
        // 因为存在hash碰撞的情况下,所以需要遍历第0索引位置的链表找到key为空的值 返回
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        //3.不存在key为空的情况，直接返回空
        return null;
    }

    /**
     * 如果该映射包含指定键的映射，则返回<tt>true</tt>。
     *
     * @param   key   The key whose presence in this map is to be tested
     * @return <tt>true</tt> if this map contains a mapping for the specified
     * key.
     */
    public boolean containsKey(Object key) {
        return getEntry(key) != null;
    }

    /**
     * 返回与HashMap中指定的键相关联的条目。如果HashMap不包含键的映射，则返回null。
     */
    final Entry<K,V> getEntry(Object key) {
        //1.size为空 说明没有数据 返回空
        if (size == 0) {
            return null;
        }
        //2.获得key的hash值
        int hash = (key == null) ? 0 : hash(key);
        //通过hash计算出table中的位置，找到table中指定链表 开始遍历
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            //3.这是判断if语句判断 涉及到一个知识点：hash值相等时 还有接着判断key值相等或者equal相等，
            // 说明如果两个对象相同，那么它们的hashCode值一定要相同 这是必要条件，不是充分条件，所以要接着判断
            // 而(k = e.key) == key 比 (key != null && key.equals(k))先判断，由于前者效率会更高，优先判断
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        //4.没有找到 返回空
        return null;
    }

    /**
     * 将此映射中的指定值与指定键关联。
     * 如果映射以前包含键的映射，旧值将被替换。
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        //1.首先判断是否还没被初始化过，如果是 则才开始真正的初始化
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        //2.1 key为空的情况
        if (key == null)
            return putForNullKey(value);
        //2.2.1 对key进行hash计算
        int hash = hash(key);
        //2.2.3 hash取余得到table索引位置
        int i = indexFor(hash, table.length);
        //2.2.4 遍历指定位置的索引的链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            //if说明参照getEntry方法说明
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        //2.3 修改次数+1  用处：保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        modCount++;
        //2.4 加入数组
        addEntry(hash, key, value, i);
        //2.5 没有旧值 直接返回空
        return null;
    }

    /**
     * 为空键卸载的put版本
     */
    private V putForNullKey(V value) {
        //1.遍历table第0索引的位置
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            //2.1.1 找到链表中key为空  ，如果找到 则将新值替换旧值 返回旧值
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        //2.2 修改次数+1 用处：保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        modCount++;
        //2.3 空键 放置于第0索引的位置 加入数组
        addEntry(0, null, value, 0);
        //2.4 返回空值
        return null;
    }

    /**
     * 构造函数和伪构造函数(clone, readObject)使用此方法而不是put。它不调整表的大小，检查是否匹配等等。它调用createEntry而不是addEntry。
     */
    private void putForCreate(K key, V value) {
        int hash = null == key ? 0 : hash(key);
        int i = indexFor(hash, table.length);

        /**
         * Look for preexisting entry for key.  This will never happen for
         * clone or deserialize.  It will only happen for construction if the
         * input Map is a sorted map whose ordering is inconsistent w/ equals.
         */
        //1.先进行查找是否已经存在对应key，找到则覆盖退出循环
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                e.value = value;
                return;
            }
        }
        //2.没有找到对应key 则进行插入指定位置的链表
        createEntry(hash, key, value, i);
    }

    private void putAllForCreate(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            putForCreate(e.getKey(), e.getValue());
    }

    /**
     * 将此映射的内容重新散列到具有更大容量的新数组中。当此映射中的键数达到其阈值时，将自动调用此方法。
     *
     * 如果当前容量是MAXIMUM_CAPACITY，此方法不会调整映射的大小，而是将阈值设置为Integer.MAX_VALUE。
     * 这具有防止以后调用的效果。
     *
     * @param newCapacity the new capacity, MUST be a power of two;
     *        must be greater than current capacity unless current
     *        capacity is MAXIMUM_CAPACITY (in which case value
     *        is irrelevant).
     */
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        //1.当旧table容量已经达到最大设定，则阈值只设定到Integer.MAX_VALUE则返回 不再进行扩容了
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
        //2.当没达到最大容量 时 则申请两倍大的旧table长度
        Entry[] newTable = new Entry[newCapacity];
        //3.将旧table数据迁移到新table中
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        //4.赋值给旧table
        table = newTable;
        //5.重新计算出阈值
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

    /**
     * 将所有条目从当前表转移到newTable。
     * 这里扩容可能会有人不明白为什么不是直接将每个旧table上的链表的头节点 赋值给 新table[i]上，而是一个个拿出来遍历，从原本的 1->2->3 变为 3->2->1？
     * 需要提一点的是扩容的目的 并不是单单将桶上的数据直接移到新数组上，而会剪短链表长度 从而达到更快获取元素的目的。
     * 那么又是如何剪短链表长度， 因为在链表循环遍历的时候 会根据新的容量进行hash会有两种情况位置可分配。
     *
     *
     */
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        //遍历旧table
        for (Entry<K,V> e : table) {
            //遍历旧table中的链表 存入新的table中
            while(null != e) {
                //保存下一次循环的 Entry<K,V>
                Entry<K,V> next = e.next;
                //判断是否需要重新hash
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                //得到e在新table中的插入位置：这里会有两种情况 1.原先位置 i 2.新位置 i+oldTable.length
                int i = indexFor(e.hash, newCapacity);
                //采用链头插入法将e插入i位置，最后得到的链表相对于原table正好是头尾相反的
                e.next = newTable[i];
                newTable[i] = e;
                //下一次循环
                e = next;
            }
        }
    }

    /**
     * 将指定映射中的所有映射复制到此映射。
     * 这些映射将替换此映射对指定映射中当前键的任何映射。
     *
     * @param m mappings to be stored in this map
     * @throws NullPointerException if the specified map is null
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        int numKeysToBeAdded = m.size();
        if (numKeysToBeAdded == 0)
            return;

        if (table == EMPTY_TABLE) {
            inflateTable((int) Math.max(numKeysToBeAdded * loadFactor, threshold));
        }

        /*
         * Expand the map if the map if the number of mappings to be added
         * is greater than or equal to threshold.  This is conservative; the
         * obvious condition is (m.size() + size) >= threshold, but this
         * condition could result in a map with twice the appropriate capacity,
         * if the keys to be added overlap with the keys already in this map.
         * By using the conservative calculation, we subject ourself
         * to at most one extra resize.
         */
        if (numKeysToBeAdded > threshold) {
            int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
            if (targetCapacity > MAXIMUM_CAPACITY)
                targetCapacity = MAXIMUM_CAPACITY;
            int newCapacity = table.length;
            while (newCapacity < targetCapacity)
                newCapacity <<= 1;
            if (newCapacity > table.length)
                resize(newCapacity);
        }

        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * 如果存在，则从此映射中删除指定键的映射。
     *
     * @param  key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V remove(Object key) {
        Entry<K,V> e = removeEntryForKey(key);
        return (e == null ? null : e.value);
    }

    /**
     * 删除并返回与HashMap中指定键相关联的条目。如果HashMap不包含此键的映射，则返回null。
     */
    final Entry<K,V> removeEntryForKey(Object key) {
        if (size == 0) {
            return null;
        }
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }

    /**
     * 使用{@code Map.Entry.equals()}进行匹配的特殊版本的EntrySet remove。
     */
    final Entry<K,V> removeMapping(Object o) {
        if (size == 0 || !(o instanceof Map.Entry))
            return null;

        Map.Entry<K,V> entry = (Map.Entry<K,V>) o;
        Object key = entry.getKey();
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            if (e.hash == hash && e.equals(entry)) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }

    /**
     * 从该映射中删除所有映射。
     * 此调用返回后映射将为空。
     */
    public void clear() {
        modCount++;
        Arrays.fill(table, null);
        size = 0;
    }

    /**
     * 如果该映射将一个或多个键映射到指定的值，则返回<tt>true</tt>。
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified value
     */
    public boolean containsValue(Object value) {
        if (value == null)
            return containsNullValue();

        Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (value.equals(e.value))
                    return true;
        return false;
    }

    /**
     * 含空参数的containsValue的特殊情况代码
     */
    private boolean containsNullValue() {
        Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (e.value == null)
                    return true;
        return false;
    }

    /**
     * Returns a shallow copy of this <tt>HashMap</tt> instance: the keys and
     * values themselves are not cloned.
     *
     * @return a shallow copy of this map
     */
    public Object clone() {
        HashMap<K,V> result = null;
        try {
            result = (HashMap<K,V>)super.clone();
        } catch (CloneNotSupportedException e) {
            // assert false;
        }
        if (result.table != EMPTY_TABLE) {
            result.inflateTable(Math.min(
                (int) Math.min(
                    size * Math.min(1 / loadFactor, 4.0f),
                    // we have limits...
                    HashMap.MAXIMUM_CAPACITY),
               table.length));
        }
        result.entrySet = null;
        result.modCount = 0;
        result.size = 0;
        result.init();
        result.putAllForCreate(this);

        return result;
    }

    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        /**
         * 每当对HashMap中已经存在的键k的put(k,v)调用覆盖条目中的值时，就会调用此方法。供LinkedHashMap使用 jdk1.8已移除
         *
         */
        void recordAccess(HashMap<K,V> m) {
        }

        /**
         * 每当条目从表中删除时，就会调用此方法。供LinkedHashMap使用 jdk1.8已移除
         */
        void recordRemoval(HashMap<K,V> m) {
        }
    }

    /**
     * 将具有指定键、值和散列码的新项添加到指定table。此方法负责在适当的情况下调整表的大小。
     *
     * 子类将重写它以改变put方法的行为。
     */
    void addEntry(int hash, K key, V value, int bucketIndex) {
        //1.判断当大小>=阈值 并且 bucketIndex位置还未存放数据
        if ((size >= threshold) && (null != table[bucketIndex])) {
            //2.1.1 扩容至原来的两倍
            resize(2 * table.length);
            //2.1.2 重新计算hash值
            hash = (null != key) ? hash(key) : 0;
            //2.1.3 对hash取余操作 获得数组下标
            bucketIndex = indexFor(hash, table.length);
        }
        //2.2 创建Entry存放进数组
        createEntry(hash, key, value, bucketIndex);
    }

    /**
     * 与addEntry相似，不同的是这个版本用于创建作为映射构造或“伪构造”(克隆、反序列化)一部分的条目。这个版本不需要担心调整表的大小。
     *
     * 子类将重写此操作以更改HashMap(Map)、clone和readObject的行为
     */
    void createEntry(int hash, K key, V value, int bucketIndex) {
        //这里使用的是头插法：将新节点插在链表的头部，此时新节点就是当前这个链表的头节点，接下来把头节点移动到数组位置即可。
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        //大小+1
        size++;
    }

    private abstract class HashIterator<E> implements Iterator<E> {
        Entry<K,V> next;        // next entry to return
        int expectedModCount;   // For fast-fail
        int index;              // current slot
        Entry<K,V> current;     // current entry

        HashIterator() {
            expectedModCount = modCount;
            if (size > 0) { // advance to first entry
                Entry[] t = table;
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
        }

        public final boolean hasNext() {
            return next != null;
        }

        final Entry<K,V> nextEntry() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            Entry<K,V> e = next;
            if (e == null)
                throw new NoSuchElementException();

            if ((next = e.next) == null) {
                Entry[] t = table;
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
            current = e;
            return e;
        }

        public void remove() {
            if (current == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            Object k = current.key;
            current = null;
            HashMap.this.removeEntryForKey(k);
            expectedModCount = modCount;
        }
    }

    private final class ValueIterator extends HashIterator<V> {
        public V next() {
            return nextEntry().value;
        }
    }

    private final class KeyIterator extends HashIterator<K> {
        public K next() {
            return nextEntry().getKey();
        }
    }

    private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {
        public Map.Entry<K,V> next() {
            return nextEntry();
        }
    }

    // Subclass overrides these to alter behavior of views' iterator() method
    Iterator<K> newKeyIterator()   {
        return new KeyIterator();
    }
    Iterator<V> newValueIterator()   {
        return new ValueIterator();
    }
    Iterator<Map.Entry<K,V>> newEntryIterator()   {
        return new EntryIterator();
    }


    // Views

    private transient Set<Map.Entry<K,V>> entrySet = null;

    /**
     * Returns a {@link Set} view of the keys contained in this map.
     * The set is backed by the map, so changes to the map are
     * reflected in the set, and vice-versa.  If the map is modified
     * while an iteration over the set is in progress (except through
     * the iterator's own <tt>remove</tt> operation), the results of
     * the iteration are undefined.  The set supports element removal,
     * which removes the corresponding mapping from the map, via the
     * <tt>Iterator.remove</tt>, <tt>Set.remove</tt>,
     * <tt>removeAll</tt>, <tt>retainAll</tt>, and <tt>clear</tt>
     * operations.  It does not support the <tt>add</tt> or <tt>addAll</tt>
     * operations.
     */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        return (ks != null ? ks : (keySet = new KeySet()));
    }

    private final class KeySet extends AbstractSet<K> {
        public Iterator<K> iterator() {
            return newKeyIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsKey(o);
        }
        public boolean remove(Object o) {
            return HashMap.this.removeEntryForKey(o) != null;
        }
        public void clear() {
            HashMap.this.clear();
        }
    }

    /**
     * Returns a {@link Collection} view of the values contained in this map.
     * The collection is backed by the map, so changes to the map are
     * reflected in the collection, and vice-versa.  If the map is
     * modified while an iteration over the collection is in progress
     * (except through the iterator's own <tt>remove</tt> operation),
     * the results of the iteration are undefined.  The collection
     * supports element removal, which removes the corresponding
     * mapping from the map, via the <tt>Iterator.remove</tt>,
     * <tt>Collection.remove</tt>, <tt>removeAll</tt>,
     * <tt>retainAll</tt> and <tt>clear</tt> operations.  It does not
     * support the <tt>add</tt> or <tt>addAll</tt> operations.
     */
    public Collection<V> values() {
        Collection<V> vs = values;
        return (vs != null ? vs : (values = new Values()));
    }

    private final class Values extends AbstractCollection<V> {
        public Iterator<V> iterator() {
            return newValueIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsValue(o);
        }
        public void clear() {
            HashMap.this.clear();
        }
    }

    /**
     * Returns a {@link Set} view of the mappings contained in this map.
     * The set is backed by the map, so changes to the map are
     * reflected in the set, and vice-versa.  If the map is modified
     * while an iteration over the set is in progress (except through
     * the iterator's own <tt>remove</tt> operation, or through the
     * <tt>setValue</tt> operation on a map entry returned by the
     * iterator) the results of the iteration are undefined.  The set
     * supports element removal, which removes the corresponding
     * mapping from the map, via the <tt>Iterator.remove</tt>,
     * <tt>Set.remove</tt>, <tt>removeAll</tt>, <tt>retainAll</tt> and
     * <tt>clear</tt> operations.  It does not support the
     * <tt>add</tt> or <tt>addAll</tt> operations.
     *
     * @return a set view of the mappings contained in this map
     */
    public Set<Map.Entry<K,V>> entrySet() {
        return entrySet0();
    }

    private Set<Map.Entry<K,V>> entrySet0() {
        Set<Map.Entry<K,V>> es = entrySet;
        return es != null ? es : (entrySet = new EntrySet());
    }

    private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return newEntryIterator();
        }
        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<K,V> e = (Map.Entry<K,V>) o;
            Entry<K,V> candidate = getEntry(e.getKey());
            return candidate != null && candidate.equals(e);
        }
        public boolean remove(Object o) {
            return removeMapping(o) != null;
        }
        public int size() {
            return size;
        }
        public void clear() {
            HashMap.this.clear();
        }
    }

    /**
     * Save the state of the <tt>HashMap</tt> instance to a stream (i.e.,
     * serialize it).
     *
     * @serialData The <i>capacity</i> of the HashMap (the length of the
     *             bucket array) is emitted (int), followed by the
     *             <i>size</i> (an int, the number of key-value
     *             mappings), followed by the key (Object) and value (Object)
     *             for each key-value mapping.  The key-value mappings are
     *             emitted in no particular order.
     */
    private void writeObject(ObjectOutputStream s)
        throws IOException
    {
        // Write out the threshold, loadfactor, and any hidden stuff
        s.defaultWriteObject();

        // Write out number of buckets
        if (table==EMPTY_TABLE) {
            s.writeInt(roundUpToPowerOf2(threshold));
        } else {
           s.writeInt(table.length);
        }

        // Write out size (number of Mappings)
        s.writeInt(size);

        // Write out keys and values (alternating)
        if (size > 0) {
            for(Map.Entry<K,V> e : entrySet0()) {
                s.writeObject(e.getKey());
                s.writeObject(e.getValue());
            }
        }
    }

    private static final long serialVersionUID = 362498820763181265L;

    /**
     * Reconstitute the {@code HashMap} instance from a stream (i.e.,
     * deserialize it).
     */
    private void readObject(ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // Read in the threshold (ignored), loadfactor, and any hidden stuff
        s.defaultReadObject();
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new InvalidObjectException("Illegal load factor: " +
                                               loadFactor);
        }

        // set other fields that need values
        table = (Entry<K,V>[]) EMPTY_TABLE;

        // Read in number of buckets
        s.readInt(); // ignored.

        // Read number of mappings
        int mappings = s.readInt();
        if (mappings < 0)
            throw new InvalidObjectException("Illegal mappings count: " +
                                               mappings);

        // capacity chosen by number of mappings and desired load (if >= 0.25)
        int capacity = (int) Math.min(
                    mappings * Math.min(1 / loadFactor, 4.0f),
                    // we have limits...
                    HashMap.MAXIMUM_CAPACITY);

        // allocate the bucket array;
        if (mappings > 0) {
            inflateTable(capacity);
        } else {
            threshold = capacity;
        }

        init();  // Give subclass a chance to do its thing.

        // Read the keys and values, and put the mappings in the HashMap
        for (int i = 0; i < mappings; i++) {
            K key = (K) s.readObject();
            V value = (V) s.readObject();
            putForCreate(key, value);
        }
    }

    // These methods are used when serializing HashSets
    int   capacity()     { return table.length; }
    float loadFactor()   { return loadFactor;   }
}

```




![think](https://img-blog.csdnimg.cn/20200612082745380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ2h1YW5nMTU3NDA1,size_16,color_FFFFFF,t_70)



## 1. 原理
HashMap采用Entry数组来存储key-value对，每一个键值对组成了一个Entry实体，Entry类实际上是一个单向的链表结构，它具有Next指针，可以连接下一个Entry实体。
![原理](https://img-blog.csdnimg.cn/20200612082745376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbG9uZ2h1YW5nMTU3NDA1,size_16,color_FFFFFF,t_70)

## 1.1 为什么用数组+链表
数组是用来确定桶的位置，利用元素的key的hash值对数组长度取模得到.
链表是用来解决hash冲突问题，当出现hash值一样的情形，就在数组上的对应位置形成一条链表

## 2. put元素的过程
对key的hashCode()做hash运算，计算index;
如果没碰撞直接放到bucket里；
如果碰撞了，以链表的形式存在buckets后；
如果节点已经存在就替换old value(保证key的唯一性)
如果bucket满了(超过load factor*current capacity)，就要resize（扩容）。

## 3. get元素的过程
对key的hashCode()做hash运算，计算index;
遍历bucket里的链表节点，去查找 key匹配的对应的Entry;

## 4. 什么条件扩容
如果HashMap存放键值对的个数满了(即超过负载因子*当前容量  默认情况当期容量的3/4)，就要扩容。
## 4.1 负载因子是0.75的好处
提高空间利用率和 减少查询成本的折中，主要是泊松分布，0.75的话碰撞最小  https://www.cnblogs.com/aspirant/p/11470928.html
## 5.容量是2次幂的好处
取模和扩容
1.取模 就是在通过key的hash与(table的长度-1)进行取余获得索引的位置。在计算机中 取余的操作还是较为消耗性能，而二进制的与操作则会非常快速。
2.扩容 当HashMap的容量达到了阈值 则会进行扩容，HashMap每次扩容都是扩容为原来的2倍；会发现规律：1.扩容后，若hash值新增参与运算的位=0，那么元素在扩容后的位置=原始位置
2.扩容后，若hash值新增参与运算的位=1，那么元素在扩容后的位置=原始位置+偏移量:(扩容后的旧位置)。 这样能有效剪短各个位置上链表的长度，从而达到更高效的获取数据

## 6. 多线程操作 出现死循环的时机 
如何出现死循环：https://www.jianshu.com/p/1e9cf0ac07f4
get与put操作，因为当某个数组上的链表变成了循环链表，在get与put操作就会遍历链表，会造成死循环

## 附录：：
## 一、jdk.map.althashing.threshold 说明

1、参数jdk.map.althashing.threshold

使用方式：-Djdk.map.althashing.threshold=5
2、作用：当hash key 是String的时候，同时hash code 算法薄弱的情况，可以降低hash值的碰撞
3、如何做到？

首先，我们都知道hashMap会根据key生成一个hash值，看代码如何生成一个key的hash值
```
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```
a、如果是String的话，就直接使用stringHash32生成hash值

b、直接调用Obejct的hashCode()方法，同时要和hashSeed 这个值进行异或操作

可以看出生成的hash值和hashSeed 这个值有着紧密的关系，但是这个值默认是0。也就是说不管HashMap存多少数据，hashSeed 都是不会变的，可以看出随着hashMap 的容量增大，hash碰撞的概率增大的可能性也就增大。如果hash值，碰撞很高的话，那么hashMap逐渐演化成链表，性能就急剧下降。


4、如何防止hashMap演化成链表？
```
static {
    String altThreshold = java.security.AccessController.doPrivileged(
        new sun.security.action.GetPropertyAction(
            "jdk.map.althashing.threshold"));

    int threshold;
    try {
        threshold = (null != altThreshold)
                ? Integer.parseInt(altThreshold)
                : ALTERNATIVE_HASHING_THRESHOLD_DEFAULT;

        // disable alternative hashing if -1
        if (threshold == -1) {
            threshold = Integer.MAX_VALUE;
        }

        if (threshold < 0) {
            throw new IllegalArgumentException("value must be positive integer.");
        }
    } catch(IllegalArgumentException failed) {
        throw new Error("Illegal value for 'jdk.map.althashing.threshold'", failed);
    }

    ALTERNATIVE_HASHING_THRESHOLD = threshold;
}
```
从代码看出`jdk.map.althashing.threshold`这个变量设置的值最终会存放在静态常量`ALTERNATIVE_HASHING_THRESHOLD`
```
final boolean initHashSeedAsNeeded(int capacity) {
    boolean currentAltHashing = hashSeed != 0;
    boolean useAltHashing = sun.misc.VM.isBooted() &&
            (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    boolean switching = currentAltHashing ^ useAltHashing;
    if (switching) {
        hashSeed = useAltHashing
            ? sun.misc.Hashing.randomHashSeed(this)
            : 0;
    }
    return switching;
}
```
当`hashMap`扩大容量时，都是调用该方法。从代码可以看出，当数组容量超过，我们设定的值`ALTERNATIVE_HASHING_THRESHOLD`且是`vm booted`，同时 `hashSeed==0`的时候，`hashSeed`的值就是用随机量，而不是固定的等于0。这样就能降低碰撞，就能降低演化成链表概率。

代码具体过程：
```
当 hashSeed==0 则 currentAltHashing=false
当 capacity < Holder.ALTERNATIVE_HASHING_THRESHOLD 则currentAltHashing =false
结果:
switching=false

当 hashSeed==0 则 currentAltHashing=false
当 capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD  则 currentAltHashing =true
结果:
switching=true


当 hashSeed !=0 则 currentAltHashing=true
当 capacity < Holder.ALTERNATIVE_HASHING_THRESHOLD  则 currentAltHashing =false
结果:
当 switching=true

当 hashSeed !=0 则 currentAltHashing=true
当 capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD  则 currentAltHashing =true
结果:
switching=false

```

5、使用场景

很少场景会用的这个值，根据我自己测试的情况，默认配置情况就碰撞率相对来说已经可以接受了，分享这个主要是看看代码是怎么实现而已。


6、总结：
```
-Djdk.map.althashing.threshold=-1:表示不做优化（不配置这个值作用一样）
-Djdk.map.althashing.threshold<0:报错

-Djdk.map.althashing.threshold=1:表示总是启用随机HashSeed
-Djdk.map.althashing.threshold>=0:便是hashMap内部的数组长度超过该值了就使用随机HashSeed，降低碰撞
```

# 二、Integer.highestOneBit((number - 1) << 1)

1、在计算机系统中，数值一律使用补码来表示和存储。主要原因是使用补码可以将符号位和其它位统一处理；同时，减法也可按照加法来处理。另 外，   两个用补码表示的数相加时，如果最高位（符号位）有进位，则进位被舍弃。
- 补码与原码的转换过程几乎相同。
    - 数值的补码表示（分两种）
        - 正数的补码：与原码相同
        - 负数的补码：符号位位1，其余位位该数绝对值的原码按位取反；然后整个数加1
    - 已知一个数的补码，求原码的操作分为两种情况
        - 如果补码的符号位“0”，表示是一个正数，所以补码就是该数的原码
        - 如果补码的符号位为“1”，表示是一个负数，求原码的操作可以是：符号位位1，其余各位取反，然后整个数加1。
2、移位运算符就是在二进制的基础上对数字进行平移。Java按照平移的方向和填充数字的规则分为三种：<<左移，>>带符号右移 和>>>无符号右移。
3、 在Java的移位运算中，byte、short和char类型移位后的结果会变成int类型，对于byte、short、char和int进行移位时，对于char、short、char和int进行移位操作时，规定实际移动的次数是移动次数和32的余数，也就是移位33次和移位1次得到的结果相同。移动long型的数值时，规定实际移动的次数是移动次数和64的余数，也就是移动65次移位1次得到相同的结果。
    （1） <<  运算规则：按二进制形式吧所有的数字向左移动对应的位数，高位移出（舍弃），低位的空位补零。
    语法格式：
         需要移位的数字<<移位的次数
         例如：4<<2 ，则是将数字4左移2位
     计算过程
         4<<2
        Java中一个int数占四个字节，那么4的二进制数字为00000000 00000000 00000000 00000100，然后把该数字左移两位。其它的数字都朝右平移两位，最后在低位（右侧）的两个空位补零。则得到的最终结果是00000000 00000000 00000000 00010000，即转换为十进制数16。
         在数字没有溢出的前提下，对于正数和负数，左移一位都相当于乘以2的1次方，左移n位就相当于乘以2的n次方。
         在溢出的前提前，则不符合这个规律。读者可以尝试输出(long)1610612736*4和1610612736<<2这两个结果进行比对。
    （2）>>运算规则：按二进制形式吧所有的数字都向右移动对应的位置，低位移出（舍弃），高位的空位补符号位，即正数补零，负数补1。
     语法格式：
         需要移位的数字>>移位的次数
         例如：-4>>2和4>>2，则是将数字 -4和4右移2位
     计算过程
         4>>2
         Java中一个int数占四个字节，同样4的二进制为00000000 00000000 00000000 00000100，然后把该数字右移两位。其它的数字都朝左平移两位，最后在高位补符号位（该数是正数，全补零），得到的结果是00000000 00000000 00000000 00000001，即使十进制的1。数学意义就是右移移位相当于除2，右移n位相当于除以2的n次方。
        4>>2
         由于负数在计算机中是以补码的形式存储的，那么-4的二进制为11111111 11111111 11111111 11111100,然后把该数字右移两位，其它数字都朝左平移两位，最后在高位补符号位（该数是负数，全补一），得到的结果是11111111 11111111 11111111 11111111（补码格式），即是十进制的-1。
    （3）>>>运算规则：按二进制形式吧所有的数字向右移动对应的位数，低位移出（舍弃），高位的空位补零。正数运算结果与带符号右移相同，对于负数来说则不同。
         对于4>>>2和-4>>>2运算，可以通过上述例子进行类推。

了解了Java的位运算之后，来看下  Integer.highestOneBit (i) 这个函数的实现代码：

```
   publicstaticinthighestOneBit(int i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
```
1、第一步的作用是把最高位1右移移位，并与原数据按位取或。那么这就使得最高位和它的下一位是连续两个1。
2、第二步的作用是把刚刚移位得到连续两个1继续右移两位并与原数据按位取或。那么这就使得最高两位和它的下两个连续位组成四个连续的1。
3、 以此类推，最终得到的i是从开始的最高位到结束全是1。并减去i不带符号的右移一位，即可得到一个int数据的最高位的值。
4、上述情况是针对于i不为零和负数的情况，如果i为零，那么得到的结果始终为零。如果i位负数，那么得到的结果始终是-2147483648。即等于Integer.MIN_VALUE。（原因在于负数的最高位始终为1，即是负数的符号位）
 
此函数的最重要理解点在与要始终把握二进制的最高位进行运算处理，那么对于函数中的右移一位、两位、四位、八和十六位就好理解了。同理，对于long类型的取最高位运算应该需要加一条语句 i|=(i>>32); 原因在于long类型在Java中是64位的。
Long类型的hightestOneBit(i)代码如下：
 
```
    public static long highestOneBit(long i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        i |= (i >> 32);
        return i - (i >>> 1);
    }
```

# 三、transfer如何理解剪短链表 提升查询效率
假设oldTable的容量是16， 假设e.hash得到的值19，二进制： 0001 0011
计算得到索引位置公式： e.hash & (length - 1) 
```
  19： 0001 0011
  15:  0000 1111
   &:  0000 0011  = 3
```
旧数据e.hash=19的位置会在索引为3的位置
那么进行扩容后 newTable的容量是原先的两倍 32，重新计算位置
```
  19： 0001 0011
  31:  0001 1111
   &:  0001 0011  = 19
```
新数据e.hash=19的位置会在索引为3的位置。可得出（i+oldTable.length）的规律

又当e.hash=3的时候，新旧位置都会锁定在3位置。
  

# 四、为什么HashMap要自己实现writeObject和readObject方法(HashMap的table为什么是transient的)
  
 首先要明确序列化的目的，将java对象序列化，一定是为了在某个时刻能够将该对象反序列化，而且一般来讲序列化和反序列化所在的机器是不同的，因为序列化最常用的场景就是跨机器的调用（把对象转化为字节流，才能进行网络传输），而序列化和反序列化的一个最基本的要求就是，反序列化之后的对象与序列化之前的对象是一致的。
HashMap中，由于Entry的存放位置是根据Key的Hash值来计算，然后存放到数组中的，对于同一个Key，在不同的JVM实现中计算得出的Hash值可能是不同的。
Hash值不同导致的结果就是：有可能一个HashMap对象的反序列化结果与序列化之前的结果不一致。即有可能序列化之前，Key=’AAA’的元素放在数组的第0个位置，而反序列化值后，根据Key获取元素的时候，可能需要从数组为2的位置来获取，而此时获取到的数据与序列化之前肯定是不同的。

所以为了避免这个问题，HashMap采用了下面的方式来解决：
将可能会造成数据不一致的元素使用transient关键字修饰，从而避免JDK中默认序列化方法对该对象的序列化操作。不序列化的包括：Entry[ ] table,size,modCount。
自己实现writeObject方法，从而保证序列化和反序列化结果的一致性。

那么，HashMap又是通过什么手段来保证序列化和反序列化数据的一致性的呢？
首先，HashMap序列化的时候不会将保存数据的数组序列化，而是将元素个数以及每个元素的Key和Value都进行序列化。
在反序列化的时候，重新计算Key和Value的位置，重新填充一个数组。


```
private void writeObject(java.io.ObjectOutputStream s) throws IOException
 
private void readObject(java.io.ObjectInputStream s) throws IOException, ClassNotFoundException
```
readObject和writeObject方法都是为了HashMap的序列化而创建的。 首先，HashMap实现了Serializable接口，这意味着该类可以被序列化，而JDK提供的对于Java对象序列化操作的类是 ObjectOutputStream，反序列化的类是 ObjectInputStream。JDK的ObjectInputStream(slotDesc.hasReadObjectMethod())、 ObjectOutputStream(slotDesc.hasWriteObjectMethod())都是会判断子类是都有自己序列