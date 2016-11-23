---
title: Java源码分析(一)-HashMap
date: 2016-09-02 16:55:34
tags:
 - Java

---

参考文章：[ java源码分析 ---- HashMap源码分析 及其 实现原理分析](http://blog.csdn.net/pandajava/article/details/42391733)

HashMap 在java集合中来说算是比较重要的一个类了。其源码也是在面试过程中经常被问的一道面试题。之前自己也大略的看过，现在抱着一起学习的态度，分析一下HashMap的源码。

### 0. 所处位置
学过Java集合的人都应该看过张图(《Java编程思想》关于容器类库的简介图)
 ![](/img/java/collection.png)

### 1. HashMap的数据结构
数据结构中有数组和链表来实现对数据的存储，但这两者基本上是两个极端。
数组
数组存储区间是连续的，占用内存严重，故空间复杂的很大。但数组的二分查找时间复杂度小，为O(1)；数组的特点是：寻址容易，插入和删除困难；
链表
链表存储区间离散，占用内存比较宽松，故空间复杂度很小，但时间复杂度很大，达O（N）。链表的特点是：寻址困难，插入和删除容易。
哈希表
那么我们能不能综合两者的特性，做出一种寻址容易，插入删除也容易的数据结构？答案是肯定的，这就是我们要提起的哈希表。哈希表（(Hash table）既满足了数据的查找方便，同时不占用太多的内容空间，使用也十分方便。
　　哈希表有多种不同的实现方法，我接下来解释的是最常用的一种方法—— 拉链法，我们可以理解为“链表的数组” ，如图：




　　从上图我们可以发现哈希表是由数组+链表组成的，一个长度为16的数组中，每个元素存储的是一个链表的头结点。那么这些元素是按照什么样的规则存储到数组中呢。一般情况是通过hash(key)%len获得，也就是元素的key的哈希值对数组长度取模得到。
比如上述哈希表中，12%16=12,28%16=12,108%16=12,140%16=12。所以12、28、108以及140都存储在数组下标为12的位置。
　　HashMap其实也是一个线性的数组实现的,所以可以理解为其存储数据的容器就是一个线性数组。这可能让我们很不解，一个线性的数组怎么实现按键值对来存取数据呢？这里HashMap有做一些处理。
　　首先HashMap里面实现一个静态内部类Entry，其重要的属性有 key , value, next，从属性key,value我们就能很明显的看出来Entry就是HashMap键值对实现的一个基础bean，我们上面说到HashMap的基础就是一个线性数组，这个数组就是Entry[]，Map里面的内容都保存在Entry[]里面。
    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    transient Entry[] table;

### 2. HashMap源码分析
 HashMap继承自AbstractMap，实现了Map接口（这些内容可以参考《Java集合类》）。来看类的定义。
 
```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
```
    Map接口定义了所有Map子类必须实现的方法。Map接口中还定义了一个内部接口Entry。

    AbstractMap也实现了Map接口，并且提供了两个实现Entry的内部类：SimpleEntry和SimpleImmutableEntry。

    定义了接口，接口中又有内部接口，然后有搞了个抽象类实现接口，抽象类里面又搞了两个内部类实现接口的内部接口，有没有点绕，为什么搞成这样呢？先不管了，先看HashMap吧。

    HashMap中定义的属性（应该都能看明白，不过还是解释一下）：
    
```
  /**
       * 默认的初始容量，必须是2的幂。
       */
      static final int DEFAULT_INITIAL_CAPACITY = 16;
      /**
       * 最大容量（必须是2的幂且小于2的30次方，传入容量过大将被这个值替换）
       */
      static final int MAXIMUM_CAPACITY = 1 << 30;
      /**
      * 默认装载因子，这个后面会做解释
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
     /**
      * 存储数据的Entry数组，长度是2的幂。看到数组的内容了，接着看数组中存的内容就明白为什么博文开头先复习数据结构了
      */
     transient Entry[] table;
     /**
      * map中保存的键值对的数量
      */
     transient int size;
     /**
      * 需要调整大小的极限值（容量*装载因子）
      */
     int threshold;
     /**
      *装载因子
      */
     final float loadFactor;
    /**
      * map结构被改变的次数
      */
     transient volatile int modCount;

```
接着是HashMap的构造方法。
    
```
/**
     *使用默认的容量及装载因子构造一个空的HashMap
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);//计算下次需要调整大小的极限值
        table = new Entry[DEFAULT_INITIAL_CAPACITY];    //根据默认容量（16）初始化table
        init();
    }
/**
     * 根据给定的初始容量的装载因子创建一个空的HashMap
     * 初始容量小于0或装载因子小于等于0将报异常 
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)//调整最大容量
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        int capacity = 1;
        //设置capacity为大于initialCapacity且是2的幂的最小值
        while (capacity < initialCapacity)  // 假设initialCapacity的值为30 则capacity的值 最终问2的5次方 32 。
            capacity <<= 1;                 //也就是说hashMap中entry[] 的初始化大小是32 而非30。
        this.loadFactor = loadFactor;
        threshold = (int)(capacity * loadFactor);
        table = new Entry[capacity];
        init();
    }
/**
     *根据指定容量创建一个空的HashMap
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);//调用上面的构造方法，容量为指定的容量，装载因子是默认值
    }
/**
     *通过传入的map创建一个HashMap，容量为默认容量（16）和(map.zise()/DEFAULT_LOAD_FACTORY)+1的较大者，装载因子为默认值
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        //调用构造方法 传入相关参数 并完成初始化
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
       
        putAllForCreate(m);
    }  
```
上面的构造方法中调用到了init()方法，最后一个方法还调用了

```
putAllForCreate(Map<? extends K, ? extends V> m)。
```
init方法是一个空方法，里面没有任何内容。（其作用为何？）
putAllForCreate看方法名就是创建的时候将传入的map全部放入新创建的对象中。该方法中还涉及到其他方法，将在后面介绍。

```
private void putAllForCreate(Map<? extends K, ? extends V> m) {  
      for (<strong>Iterator<? extends Map.Entry<? extends K, ? extends V>> i =<u> m.entrySet().iterator()</u>; i.hasNext();</strong> ) {  
          <strong>Map.Entry<? extends K, ? extends V> e = i.next();</strong>  
          <u>putForCreate(e.getKey(), e.getValue());</u>  
      }  
  }
  
``` 
  
先看初始化table时均使用了Entry，这是HashMap的一个内部类，实现了Map接口的内部接口Entry。下面给出Map.Entry接口及HashMap.Entry类的内容。Map.Entry接口定义的方法
    
```
 K getKey();//获取Key
 V getValue();//获取Value
 V setValue();//设置Value，至于具体返回什么要看具体实现
 boolean equals(Object o);//定义equals方法用于判断两个Entry是否相同
 int hashCode();//定义获取hashCode的方法

```
 HashMap.Entry类的具体实现

```
  static class Entry<K,V> implements Map.Entry<K,V> {
          final K key;
          V value;
          Entry<K,V> next;//对下一个节点的引用（看到链表的内容，结合定义的Entry数组，是不是想到了哈希表的拉链法实现？！）
          final int hash;//哈希值
  
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
             return oldValue;//返回的是之前的Value
         }
 
         public final boolean equals(Object o) {
             if (!(o instanceof Map.Entry))//先判断类型是否一致
                    return false;
             Map.Entry e = (Map.Entry)o;
             Object k1 = getKey();
             Object k2 = e.getKey();
             // Key相等且Value相等则两个Entry相等
             if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                 Object v1 = getValue();
                 Object v2 = e.getValue();
                 if (v1 == v2 || (v1 != null && v1.equals(v2)))
                     return true;
             }
             return false;
         }
         // hashCode是Key的hashCode和Value的hashCode的异或的结果
         public final int hashCode() {
             return (key==null   ? 0 : key.hashCode()) ^
                   (value==null ? 0 : value.hashCode());
         }
         // 重写toString方法，是输出更清晰
         public final String toString() {
             return getKey() + "=" + getValue();
        }
 
        /**
          *当调用put(k,v)方法存入键值对时，如果k已经存在，则该方法被调用（为什么没有内容？）
          */
        void recordAccess(HashMap<K,V> m) {
        }

        /**
          * 当Entry被从HashMap中移除时被调用（为什么没有内容？）
          */
         void recordRemoval(HashMap<K,V> m) {
         }
     }
```
看完属性和构造方法，接着看HashMap中的其他方法，一个个分析，从最常用的put和get说起吧。
put()该方法 返回与 key 关联的旧值；如果key 没有任何映射关系，则返回null。
     
```
  public V put(K key, V value) {
          if (key == null)
              return putForNullKey(value);

          int hash = hash(key.hashCode());
          int i = indexFor(hash, table.length);  //获取应该存放的索引值

          for (Entry<K,V> e = table[i]; e != null; e = e.next) {  //校验key值之前是否存在，存在则替换并返回先前的值
             Object k;
             if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                 V oldValue = e.value;
                 e.value = value;
                 e.recordAccess(this);
                 return oldValue;
             }
         }

         // 该key之前不存在 则 执行 添加操作
         modCount++;
         addEntry(hash, key, value, i);   //四个参数 hash值  key  value  i索引
         return null;
     }
    当存入的key是null的时候将调用putForNUllKey方法，暂时将这段逻辑放一边，看key不为null的情况。

```
先调用了hash(int h)方法获取了一个hash值。
    
```
 static int hash(int h) {
         // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
         // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
         return h ^ (h >>> 7) ^ (h >>> 4);
    }
```
这个方法的主要作用是防止质量较差的哈希函数带来过多的冲突（碰撞）问题。Java中int值占4个字节，即32位。根据这32位值进行移位、异或运算得到一个值。

```
 static int indexFor(int h, int length) {
         return h & (length-1);
     }
```
indexFor返回hash值和table数组长度减1的与运算结果，获取索引位置。
为什么使用的是length-1？因为这样可以保证结果的最大值是length-1，不会产生数组越界问题。获取索引位置之后做了什么？探测table[i]所在的链表，所发现key值与传入的key值相同的对象，则替换并返回oldValue。（key值之前存在，替换并且返回旧值）若找不到，则通过addEntry(hash,key,value,i)添加新的对象。 （key值之前不存在，添加addEntry方法 ，并返回null）

来看addEntry(hash,key,value,i)方法。

```
void addEntry(int hash, K key, V value, int bucketIndex) {
         Entry<K,V> e = table[bucketIndex];   //bucketIndex位置的元素先保存起来（新加的entry要占据这个位置）
         table[bucketIndex] = new Entry<K,V>(hash, key, value, e);//将新的值存到bucketIndex处，该值的next指向 刚刚保存的e元素
         //检测是不是需要扩充容量 （2倍2倍的扩容）
          if (size++ >= threshold)   // threshold默认值 是  16 * 0.75 = 12             resize(2 * table.length);
  }
```
这就是在一个链表头部插入一个节点的过程。获取table[i]的对象e，将table[i]的对象修改为新增对象，让新增对象的next指向e。之后判断size是否到达了需要扩充table数组容量的界限并让size自增1，如果达到了则调用resize(int capacity)方法将数组容量拓展为原来的两倍。

```
 void resize(int newCapacity) {
          Entry[] oldTable = table;
          int oldCapacity = oldTable.length;
          // 这个if块表明，如果容量已经到达允许的最大值，即MAXIMUN_CAPACITY，则不再拓展容量，而将装载拓展的界限值设为计算机允许的最大值。
         // 不会再触发resize方法，而是不断的向map中添加内容，即table数组中的链表可以不断变长，但数组长度不再改变
         if (oldCapacity == MAXIMUM_CAPACITY) {  //MAXIMUM_CAPACITY  1073741824 [0x40000000]
             threshold = Integer.MAX_VALUE;
             return;
          }
         // 创建新数组，容量为指定的容量
         Entry[] newTable = new Entry[newCapacity];
         transfer(newTable);
         table = newTable;
         // 设置下一次需要调整数组大小的界限
         threshold = (int)(newCapacity * loadFactor);
     }
```
 结合上面给出的注释，调整数组容量的内容仅剩下将原table中的内容复制到newTable中并将newTable赋值给给table变量。即上面代码中的“transfer(newTable);table = newTable;”。
来看transfer(Entry[] newTable)方法。

```
  void transfer(Entry[] newTable) {
          // 保留原数组的引用到src中，
          Entry[] src = table;
          // 新容量使新数组的长度
          int newCapacity = newTable.length;
          // 遍历原数组
         for (int j = 0; j < src.length; j++) {
              // 获取元素e （此处的e是单链表的头结点）
              Entry<K,V> e = src[j];
             if (e != null) {
                 // 将原数组中的元素置为null
                 src[j] = null;
                 // 遍历原数组中j位置指向的链表
                 do {
                     Entry<K,V> next = e.next;
                     // 根据新的容量计算e在新数组中的位置
                     int i = indexFor(e.hash, newCapacity);
                     // 将e插入到newTable[i]指向的链表的头部
                     e.next = newTable[i];
                     newTable[i] = e;
                    e = next;
                 } while (e != null);
             }
         }
     }
```

这样处理效果 原来的 单链表如果 顺序是  （头） a b c d e  （尾）通过  transfer方法之后的顺序是  (头)  e  d c  b a （尾）  因为后插入的在 链表的头部。从上面的代码可以看出，HashMap之所以不能保持元素的顺序有以下几点原因：

    第一，插入元素的时候对元素进行哈希处理，不同元素分配到table的不同位置；
    第二，容量拓展的时候又进行了hash处理；
    第三，复制原表内容的时候链表被倒置。

一个put方法带出了这么多内容，接着看看putAll吧。

```
  public void putAll(Map<? extends K, ? extends V> m) {
          int numKeysToBeAdded = m.size();
          if (numKeysToBeAdded == 0)
              return;
          // 为什么判断条件是numKeysToBeAdded，不是(numKeysToBeAdded+table.length)>threshold???
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
 
         for (Iterator<? extends Map.Entry<? extends K, ? extends V>> i = m.entrySet().iterator(); i.hasNext(); ) {
            Map.Entry<? extends K, ? extends V> e = i.next();
             put(e.getKey(), e.getValue());
         }
     }
```

先回答上面的问题：为什么判断条件是numKeysToBeAdded，不是(numKeysToBeAdded+table.length)>threshold 
这是一种保守的做法，明显地，我们应该在(numKeysToBeAdded+table.length)>threshold的时候去拓展容量，但是考虑到将被添加的元素可能会有Key与原本存在的Key相同的情况，所以采用保守的做法，避免拓展到过大的容量。 接着是遍历m中的内容，然后调用put方法将元素添加到table数组中。遍历的时候涉及到了entrySet方法，这个方法定义在Map接口中，HashMap中也有实现，后面会解释HashMap的这个方法，其它Map的实现暂不解释。

	下面介绍在put方法中被调用到的putForNullKey方法。

```

 private V putForNullKey(V value) {
         for (Entry<K,V> e = table[0]; e != null; e = e.next) {
             if (e.key == null) {
                 V oldValue = e.value;
                 e.value = value;
                 e.recordAccess(this);
                  return oldValue;
             }
         }
         modCount++;
         addEntry(0, null, value, 0); // null 的 hash值是0 。 索引位置 index也是 0
        return null;
     }
```

这是一个私有方法，在put方法中被调用。它首先遍历table数组，如果找到key为null的元素，则替换元素值并返回oldValue；否则通过addEntry方法添加元素，之后返回null。还记得上面构造方法中调用到的putAllForCreate吗？一口气将put操作的方法看完吧。

```
 private void putAllForCreate(Map<? extends K, ? extends V> m) {
         for (Iterator<? extends Map.Entry<? extends K, ? extends V>> i = m.entrySet().iterator(); i.hasNext(); ) {
             Map.Entry<? extends K, ? extends V> e = i.next();
             putForCreate(e.getKey(), e.getValue());
         }
     }
```
先将遍历的过程放在一边，因为它同样涉及到了entrySet()方法。剩下的代码很简单，只是调用putForCreate方法逐个元素加入。

```
 private void putForCreate(K key, V value) {
          int hash = (key == null) ? 0 : hash(key.hashCode());
          int i = indexFor(hash, table.length);
          for (Entry<K,V> e = table[i]; e != null; e = e.next) {
              Object k;
              if (e.hash == hash &&
                  ((k = e.key) == key || (key != null && key.equals(k)))) {
                  e.value = value;
                  return;
             }
         }
         createEntry(hash, key, value, i);
     }
```
     
 该方法先计算需要添加的元素的hash值和在table数组中的索引i。接着遍历table[i]的链表，若有元素的key值与传入key值相等，则替换value，       结束方法。若不存在key值相同的元素，则调用createEntry创建并添加元素。

```
void createEntry(int hash, K key, V value, int bucketIndex) {
          Entry<K,V> e = table[bucketIndex];
         table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
         size++;
     }
```

这个方法的内容就不解释了，上面都解释过。 至此所有put相关操作都解释完毕了。put之外，另一个常用的操作就是get，下面就来看get方法。

```
 public V get(Object key) {
          if (key == null)
              return getForNullKey();
          int hash = hash(key.hashCode());
          for (Entry<K,V> e = table[indexFor(hash, table.length)];e != null; e = e.next) {
              Object k;
              if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                 return e.value;
         }
         return null;
     }
 ```
     
 该方法分为key为null和不为null两块。先看不为null的情况。先获取key的hash值，之后通过hash值及table.length获取key对应的table数组的索引，遍历索引的链表，所找到key相同的元素，则返回元素的value，否者返回null。不为null的情况调用了getForNullKey()方法。

1 private V getForNullKey() {
2         for (Entry<K,V> e = table[0]; e != null; e = e.next) {
3             if (e.key == null)
4                 return e.value;
5         }
6         return null;
7     }
    这是一个私有方法，只在get中被调用。该方法判断table[0]中的链表是否包含key为null的元素，包含则返回value，不包含则返回null。为什么是遍历table[0]的链表？因为key为null的时候获得的hash值都是0。
添加（put）和获取（get）都结束了，接着看如何判断一个元素是否存在。
HashMap没有提供判断元素是否存在的方法，只提供了判断Key是否存在及Value是否存在的方法，分别是containsKey(Object key)、containsValue(Object value)。
containsKey(Object key)方法很简单，只是判断getEntry(key)的结果是否为null，是则返回false，否返回true。

```
 public boolean containsKey(Object key) {
          return getEntry(key) != null;
      }
  final Entry<K,V> getEntry(Object key) {
          int hash = (key == null) ? 0 : hash(key.hashCode());
          for (Entry<K,V> e = table[indexFor(hash, table.length)]; e != null;e = e.next) {
              Object k;
             if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
                 return e;
         }
         return null;
     }
```
getEntry(Object key)也没什么内容，只是根据key对应的hash值计算在table数组中的索引位置，然后遍历该链表判断是否存在相同的key值。

```

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
 private boolean containsNullValue() {
     Entry[] tab = table;
         for (int i = 0; i < tab.length ; i++)
             for (Entry e = tab[i] ; e != null ; e = e.next)
                 if (e.value == null)
                     return true;
     return false;
     }
```

判断一个value是否存在比判断key是否存在还要简单，就是遍历所有元素判断是否有相等的值。这里分为两种情况处理，value为null何不为null的情况，但内容差不多，只是判断相等的方式不同。这个判断是否存在必须遍历所有元素，是一个双重循环的过程，因此是比较耗时的操作。接着看HashMap中“删除”相关的操作，有remove(Object key)和clear()两个方法。

    remove(Object key)

```
public V remove(Object key) {
         Entry<K,V> e = removeEntryForKey(key);
         return (e == null ? null : e.value);
     }
```

看这个方法，removeEntryKey(key)的返回结果应该是被移除的元素，如果不存在这个元素则返回为null。remove方法根据removeEntryKey返回的结果e是否为null返回null或e.value。

    removeEntryForKey(Object key)
```
  final Entry<K,V> removeEntryForKey(Object key) {
          int hash = (key == null) ? 0 : hash(key.hashCode());
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

```
    上面的这个过程就是先找到table数组中对应的索引，接着就类似于一般的链表的删除操作，而且是单向链表删除节点，很简单。在C语言中就是修改指针，这个例子中就是将要删除节点的前一节点的next指向删除被删除节点的next即可。

clear()

```
 public void clear() {
         modCount++;
         Entry[] tab = table;
         for (int i = 0; i < tab.length; i++)
             tab[i] = null;
         size = 0;
    }
```

clear()方法删除HashMap中所有的元素，这里就不用一个个删除节点了，而是直接将table数组内容都置空，这样所有的链表都已经无法访问，Java的垃圾回收机制会去处理这些链表。table数组置空后修改size为0。这里为什么不直接操作table而是通过tab呢？希望有知道的大侠指点一二。主要方法看的差不多了，接着看一个上面提到了好几次但是都搁在一边没有分析的方法：entrySet()。 

entrySet()

```
	public Set<Map.Entry<K,V>> entrySet() {
     return entrySet0();
   }
 
    private Set<Map.Entry<K,V>> entrySet0() {
         Set<Map.Entry<K,V>> es = entrySet;
         return es != null ? es : (entrySet = new EntrySet());
    }
```

为什么会有这样的方法，只是调用了一下entrySet0，而且entrySet0的名称看着就很奇怪。再看entrySet0方法中为什么不直接return entrySet!=null?entrySet:(entrySet = new EntrySet)呢？上面的疑问还没解开，但是先看entrySet这个属性吧，在文章开头的属性定义中并没有给出这个属性，下面先看一下它的定义：

```
private transient Set<Map.Entry<K,V>> entrySet = null;

```
它是一个内容为Map.Entry<K,V>的Set。看看在哪些地方往里面添加了元素。 为什么上面的那句话我要把它标成红色？因为这是一个陷阱，在看代码的时候我就陷进去了。仔细看EntrySet这个类。

```

private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
         public Iterator<Map.Entry<K,V>> iterator() {
             return new EntryIterator();
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

```
看到了什么？这个类根本就没属性，它只是个代理。因为它内部类，可以访问外部类的内容，debug的时候能看到的属性都是继承或者外部类的属性，输出的时候其实也是调用到了父类的toString方法将HashMap中的内容输出了。

    keySet()

1 public Set<K> keySet() {
2         Set<K> ks = keySet;
3         return (ks != null ? ks : (keySet = new KeySet()));
4     }
    是不是和entrySet0()方法很像！

 1 private final class KeySet extends AbstractSet<K> {
 2         public Iterator<K> iterator() {
 3             return newKeyIterator();
 4         }
 5         public int size() {
 6             return size;
 7         }
 8         public boolean contains(Object o) {
 9             return containsKey(o);
10         }
11         public boolean remove(Object o) {
12             return HashMap.this.removeEntryForKey(o) != null;
13         }
14         public void clear() {
15             HashMap.this.clear();
16         }
17     }
复制代码
    同样是个代理类，contains、remove、clear方法都是调用的HashMap的方法。 

 

    values()

 1 public Collection<V> values() {
 2         Collection<V> vs = values;
 3         return (vs != null ? vs : (values = new Values()));
 4     }
 5 
 6     private final class Values extends AbstractCollection<V> {
 7         public Iterator<V> iterator() {
 8             return newValueIterator();
 9         }
10         public int size() {
11             return size;
12         }
13         public boolean contains(Object o) {
14             return containsValue(o);
15         }
16         public void clear() {
17             HashMap.this.clear();
18         }
19     }
    values()方法也一样是代理。只是Values类继承自AbstractCollention类，而不是AbstractSet。

--------------------------------------------------------------------------------------------------------------------

1. HashMap概述：

　　HashMap是基于哈希表的Map接口的非同步实现（Hashtable跟HashMap很像，唯一的区别是Hashtalbe中的方法是线程安全的，也就是同步的）。此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

2. HashMap的数据结构：

　　在java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表的数组”的数据结构，每个元素存放链表头结点的数组，即数组和链表的结合体。



　　从上图中可以看出，HashMap底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个HashMap的时候，就会初始化一个数组。源码如下：

/**
 * The table, resized as necessary. Length MUST Always be a power of two.
 */
transient Entry[] table;

static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    final int hash;
    ……
}
　　可以看出，Entry就是数组中的元素，每个 Map.Entry 其实就是一个key-value对，它持有一个指向下一个元素的引用，这就构成了链表。

3.    HashMap的存取实现：

  1) 存储：

public V put(K key, V value) {
    // HashMap允许存放null键和null值。
    // 当key为null时，调用putForNullKey方法，将value放置在数组第一个位置。
    if (key == null)
        return putForNullKey(value);
    // 根据key的hashCode重新计算hash值。
    int hash = hash(key.hashCode());
    // 搜索指定hash值所对应table中的索引。
    int i = indexFor(hash, table.length);
    // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素。
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    // 如果i索引处的Entry为null，表明此处还没有Entry。
    // modCount记录HashMap中修改结构的次数
    modCount++;
    // 将key、value添加到i索引处。
    addEntry(hash, key, value, i);
    return null;
}
　　从上面的源代码中可以看出：当我们往HashMap中put元素的时候，先根据key的hashCode重新计算hash值，根据hash值得到这个元素在数组中的位置（即下标），如果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，新加入的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。

　　addEntry(hash, key, value, i)方法根据计算出的hash值，将key-value对放在数组table的 i 索引处。addEntry 是HashMap 提供的一个包访问权限的方法（就是没有public，protected，private这三个访问权限修饰词修饰，为默认的访问权限，用default表示，但在代码中没有这个default），代码如下：

void addEntry(int hash, K key, V value, int bucketIndex) {
    // 获取指定 bucketIndex 索引处的 Entry 
    Entry<K,V> e = table[bucketIndex];
    // 将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    // 如果 Map 中的 key-value 对的数量超过了极限
    if (size++ >= threshold)
    // 把 table 对象的长度扩充到原来的2倍。
        resize(2 * table.length);
}
　　当系统决定存储HashMap中的key-value对时，完全没有考虑Entry中的value，仅仅只是根据key来计算并决定每个Entry的存储位置。我们完全可以把 Map 集合中的 value 当成 key 的附属，当系统决定了 key 的存储位置之后，value 随之保存在那里即可。

　　hash(int h)方法根据key的hashCode重新计算一次散列。此算法加入了高位计算，防止低位不变，高位变化时，造成的hash冲突。

static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
　　我们可以看到在HashMap中要找到某个元素，需要根据key的hash值来求得对应数组中的位置。如何计算这个位置就是hash算法。前面说过HashMap的数据结构是数组和链表的结合，所以我们当然希望这个HashMap里面的 元素位置尽量的分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用hash算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，而不用再去遍历链表，这样就大大优化了查询的效率。

　　对于任意给定的对象，只要它的 hashCode() 返回值相同，那么程序调用 hash(int h) 方法所计算得到的 hash 码值总是相同的。我们首先想到的就是把hash值对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，“模”运算的消耗还是比较大的，在HashMap中是这样做的：调用 indexFor(int h, int length) 方法来计算该对象应该保存在 table 数组的哪个索引处。indexFor(int h, int length) 方法的代码如下：

static int indexFor(int h, int length) {
    return h & (length-1);
}
　　这个方法非常巧妙，它通过 h & (table.length -1) 来得到该对象的保存位，而HashMap底层数组的长度总是 2 的n 次方，这是HashMap在速度上的优化。在 HashMap 构造器中有如下代码：

int capacity = 1;
    while (capacity < initialCapacity)
        capacity <<= 1;
　　这段代码保证初始化时HashMap的容量总是2的n次方，即底层数组的长度总是为2的n次方。

　　当length总是 2 的n次方时，h& (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。

　　这看上去很简单，其实比较有玄机的，我们举个例子来说明：

　　假设数组长度分别为15和16，优化后的hash码分别为8和9，那么&运算后的结果如下：

       h & (table.length-1)                     hash                             table.length-1

       8 & (15-1)：                                 0100                   &              1110                   =                0100

       9 & (15-1)：                                 0101                   &              1110                   =                0100

      -----------------------------------------------------------------------------------------------------------------------

       8 & (16-1)：                                 0100                   &              1111                   =                0100

       9 & (16-1)：                                 0101                   &              1111                   =                0101

      -----------------------------------------------------------------------------------------------------------------------

　　从上面的例子中可以看出：当8、9两个数和(15-1)2=(1110)进行“与运算&”的时候，产生了相同的结果，都为0100，也就是说它们会定位到数组中的同一个位置上去，这就产生了碰撞，8和9会被放到数组中的同一个位置上形成链表，那么查询的时候就需要遍历这个链 表，得到8或者9，这样就降低了查询的效率。同时，我们也可以发现，当数组长度为15的时候，hash值会与(15-1)2=(1110)进行“与运算&”，那么最后一位永远是0，而0001，0011，0101，1001，1011，0111，1101这几个位置永远都不能存放元素了，空间浪费相当大，更糟的是这种情况中，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率！

　　而当数组长度为16时，即为2的n次方时，2n-1得到的二进制数的每个位上的值都为1（比如(24-1)2=1111），这使得在低位上&时，得到的和原hash的低位相同，加之hash(int h)方法对key的hashCode的进一步优化，加入了高位计算，就使得只有相同的hash值的两个值才会被放到数组中的同一个位置上形成链表。

　　所以说，当数组长度为2的n次幂的时候，不同的key算得得index相同的几率较小，那么数据在数组上分布就比较均匀，也就是说碰撞的几率小，相对的，查询的时候就不用遍历某个位置上的链表，这样查询效率也就较高了。

　　根据上面 put 方法的源代码可以看出，当程序试图将一个key-value对放入HashMap中时，程序首先根据该 key的 hashCode() 返回值决定该 Entry 的存储位置：如果两个 Entry 的 key 的 hashCode() 返回值相同，那它们的存储位置相同。如果这两个 Entry 的 key 通过 equals 比较返回 true，新添加 Entry 的 value 将覆盖集合中原有Entry 的 value，但key不会覆盖。如果这两个 Entry 的 key 通过 equals 比较返回 false，新添加的 Entry 将与集合中原有 Entry 形成 Entry 链，而且新添加的 Entry 位于 Entry 链的头部——具体说明继续看 addEntry() 方法的说明。

  2) 读取：

public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
        e != null;
        e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
　　有了上面存储时的hash算法作为基础，理解起来这段代码就很容易了。从上面的源代码中可以看出：从HashMap中get元素时，首先计算key的hashCode，找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。

  3) 归纳起来简单地说，HashMap 在底层将 key-value 当成一个整体进行处理，这个整体就是一个 Entry 对象。HashMap 底层采用一个 Entry[] 数组来保存所有的 key-value 对，当需要存储一个 Entry 对象时，会根据hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；当需要取出一个Entry时，也会根据hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。

4. HashMap的resize（rehash）：

　　当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，这是一个常用的操作，而在HashMap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。

　　那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。也就是说，默认情况下，数组大小为16，那么当HashMap中元素个数超过16*0.75=12（这个值就是代码中的threshold值，也叫做临界值）的时候，就把数组的大小扩展为 2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。

HashMap扩容的代码如下所示：

//HashMap数组扩容
          void resize(int newCapacity) {
                Entry[] oldTable = table;
                int oldCapacity = oldTable.length;
                //如果当前的数组长度已经达到最大值，则不在进行调整
                if (oldCapacity == MAXIMUM_CAPACITY) {
                    threshold = Integer.MAX_VALUE;
                    return;
                }
                //根据传入参数的长度定义新的数组
                Entry[] newTable = new Entry[newCapacity];
                //按照新的规则，将旧数组中的元素转移到新数组中
                transfer(newTable);
                table = newTable;
                //更新临界值
                threshold = (int)(newCapacity * loadFactor);
            }

          //旧数组中元素往新数组中迁移
            void transfer(Entry[] newTable) {
                //旧数组
                Entry[] src = table;
                //新数组长度
                int newCapacity = newTable.length;
                //遍历旧数组
                for (int j = 0; j < src.length; j++) {
                    Entry<K,V> e = src[j];
                    if (e != null) {
                        src[j] = null;
                        do {
                            Entry<K,V> next = e.next;
                            int i = indexFor(e.hash, newCapacity);
                            e.next = newTable[i];
                            newTable[i] = e;
                            e = next;
                        } while (e != null);
                    }
                }
            }
5.HashMap的性能参数：

HashMap 包含如下几个构造器：

   HashMap()：构建一个初始容量为 16，负载因子为 0.75 的 HashMap。
   HashMap(int initialCapacity)：构建一个初始容量为 initialCapacity，负载因子为 0.75 的 HashMap。
   HashMap(int initialCapacity, float loadFactor)：以指定初始容量、指定的负载因子创建一个 HashMap。    initialCapacity：HashMap的最大容量，即为底层数组的长度。 loadFactor：负载因子loadFactor定义为：散列表的实际元素数目(n)/ 散列表的容量(m)。
　　负载因子衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。

　　HashMap的实现中，通过threshold字段来判断HashMap的最大容量：

threshold = (int)(capacity * loadFactor);  
　　结合负载因子的定义公式可知，threshold就是在此loadFactor和capacity对应下允许的最大元素数目，超过这个数目就重新resize，以降低实际的负载因子（也就是说虽然数组长度是capacity，但其扩容的临界值确是threshold）。默认的的负载因子0.75是对空间和时间效率的一个平衡选择。当容量超出此最大容量时， resize后的HashMap容量是容量的两倍：

if (size++ >= threshold)   
    resize(2 * table.length); 
6.Fail-Fast机制：

　　我们知道java.util.HashMap不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。（这个在core java这本书中也有提到。）

　　这一策略在源码中的实现是通过modCount域，modCount顾名思义就是修改次数，对HashMap内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。

HashIterator() {
    expectedModCount = modCount;
    if (size > 0) { // advance to first entry
    Entry[] t = table;
    while (index < t.length && (next = t[index++]) == null)
        ;
    }
}
　　在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map：

　　注意到modCount声明为volatile，保证线程之间修改的可见性。

    （volatile之所以线程安全是因为被volatile修饰的变量不保存缓存，直接在内存中修改，因此能够保证线程之间修改的可见性）。

final Entry<K,V> nextEntry() {   
    if (modCount != expectedModCount)   
        throw new ConcurrentModificationException();
在HashMap的API中指出：

　　由所有HashMap类的“collection 视图方法”所返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，而不保证在将来不确定的时间发生任意不确定行为的风险。

　　注意，迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。