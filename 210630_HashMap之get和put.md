
>- ## HashMap----get && put
>  
- ## 基本结构
  - 数组加链表
- ## 基本变量
  - 初始容量，默认16
    ```java 
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16 
    ```
  - 最大容量，默认2的31次方
    ```java 
    static final int MAXIMUM_CAPACITY = 1 << 30;
    ```
  - 装载因子，默认0.75f
    ```java 
    static final float DEFAULT_LOAD_FACTOR = 0.75f; 
    ```
  - 当一个元素被添加到至少TREEIFY_THRESHOLD个节点的桶中，桶中链表将被转化为树形结构，默认8
    ```java 
    static final int TREEIFY_THRESHOLD = 8; 
    ```
  - 当一个桶中的数形结构节点数小于UNTREEIFY_THRESHOLD时，将树型结构转化为链表
    ```java 
    static final int UNTREEIFY_THRESHOLD = 6;
    ```
  - 当散列表容量小于该阈值，即使桶中节点的数量超过了TREEIFY_THRESHOLD，也不会进行树化，只会进行扩容操作
    ```java 
    static final int MIN_TREEIFY_CAPACITY = 64; 
    ```
- ## get与put
   - get(Object key)
     ```java
     public V get(Object key) {
        Node<K,V> e;
        // 调用 getNode()方法，将key的hash值与key传入
        return (e = getNode(hash(key), key)) == null ? null : e.value;
      }
     
     final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 判断table是否为空 或者 通过hash与table长度位与得到的地址的节点是否为空 ，是则直接返回null
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 比较第一个节点的hash、key是否相等，相等直接返回
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 将第一个节点的下个节点指向e，判断是否为空，是则直接返回null
            if ((e = first.next) != null) {
                // 如果第一个节点属于树型结构，调用树型结构的getTreeNode方法
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 遍历链表
                do {
                    // 找到hash相等并且key相等，则返回该节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
      }
     ```
   - put(K key, V value)
     ```java
     public V put(K key, V value) {
        // 调用putVal方法，传入key的hash值与key、value
        return putVal(hash(key), key, value, false, true);
     }
     
     final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 如果table为空，调用resize()方法初始化扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 通过hash与table长度位与的到的地址的节点为空，则直接新建节点并赋值
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 如果位与后地址的节点hash相等，并且key相等，则直接赋值e
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果第一个节点属于树型结构节点，则调用树型结构的putTreeVal方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 遍历该链表，如果存在相同key，替换value
            else {
                for (int binCount = 0; ; ++binCount) {
                    // 遍历到最后没有找到
                    if ((e = p.next) == null) {
                        // 新建节点，链表最后一个节点指向新节点
                        p.next = newNode(hash, key, value, null);
                        // 如果到达TREEIFY_THRESHOLD大小，调用treeifyBin从链表转化为树型结构
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 找到了hash相等并且key相等的节点e，跳出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // e为空则是新建，e不为空则是替换节点value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                // 更新value
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // hashMap的afterNodeAccess是空方法，此处是为了LinkedHashMap而存在
                afterNodeAccess(e);
                // 返回旧值，put如果存在key节点，则返回旧节点的value，不存在则返回null
                return oldValue;
            }
        }
        // 修改次数加1
        ++modCount;
        // 如果达到扩容条件，调用resize方法扩容
        if (++size > threshold)
            resize();
        // hashMap的afterNodeInsertion是空方法，此处是为了LinkedHashMap而存在
        afterNodeInsertion(evict);
        return null;
     }
     ```