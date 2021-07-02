
>- ## HashMap的resize()
>  
- ## resize()
  - resize()
  ```java
  final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        // 旧表长度
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 旧表阈值
        int oldThr = threshold;
        int newCap, newThr = 0;
        // 旧表长度大于0
        if (oldCap > 0) {
            // 如果旧表长度已经大于最大容量，赋值阈值为最大整型，将不再进行扩容，返回旧表
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            /**
             *新表长度赋值为旧表的两倍，当新表长度小于最大容量、旧表长度大于等于默认容量时，
             * 赋值新表阈值为旧表阈值的两倍
             */
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        /**
         * 从构造方法中
         * 如果没有指定initialCapacity, 则不会给threshold赋值, 该值被初始化为0
         * 如果指定了initialCapacity, 该值被初始化成大于initialCapacity的最小的2的次幂
         * 这里这种情况指的是原table为空，并且在初始化的时候指定了容量，
         * 则用threshold作为table的实际大小
         */
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        // 构造方法中没有指定容量，则使用默认值
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 计算指定了initialCapacity情况下的新的threshold
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 旧表不为空
        if (oldTab != null) {
            // 遍历旧表每个数组节点
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                // 将头节点赋值e，e不为空
                if ((e = oldTab[j]) != null) {
                    // 旧表该头节点赋值空释放，现在已有e引用头节点
                    oldTab[j] = null;
                    // e的下一节点为空，则直接位于计算出新地址赋值到新表数组位置
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // e为红黑树结构直接调用树节点split方法(这里不做阐述)
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 开始链表移动扩容
                    else { // preserve order
                        // 旧头节点，旧尾节点
                        Node<K,V> loHead = null, loTail = null;
                        // 新头节点，新尾节点
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            // 将e的下一节点赋值next
                            next = e.next;
                            // 计算hash值与旧表大小位于是否为0，为0则代表e.hash与oldCap最高位与为0，不需要做移动
                            if ((e.hash & oldCap) == 0) {
                                // 将旧头节点赋值为e，这里只会在第一次的时候执行一次，目的就是找到该链表的头节点
                                if (loTail == null)
                                    loHead = e;
                                /**
                                 *将旧尾节点的下一节点赋值e，此处赋值会改变旧头节点的下一节点，
                                 *目的就是使符合不需要移动的节点形成新的链表，头节点为loHead
                                 */
                                else
                                    loTail.next = e;
                                // 将旧尾节点赋值e，目的就是找到此时链表操作到的最后一个节点
                                loTail = e;
                            }
                            else {
                                // 与旧头、尾节点操作一致
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        /**
                         *旧尾节点不为空，将旧尾节点赋值空释放，
                         *将该地址头节点赋值为不需要移动位置的旧链表头节点loHead
                         */
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        /**
                         *新尾节点不为空，将新尾节点赋值空释放，将该地址节点赋值为需要移动位置的新链表节点hiHead，
                         *此处地址为什么要加上旧表大小，原因是该链表节点的hash值与新表大小位与的时候，
                         *新表大小的最高位上位与是为1的，而此时计算最高位为1时的值是旧表的大小值
                         */
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
  ```
  - 为什么`newTab[j + oldCap] = hiHead;`需要加上oldCap
  ```text
  hash值26，二进制11010
  旧表大小16，二进制10000，与hash值位与寻址的二进制1111
  新表大小32，二进制100000，与hash值位与寻址的二进制11111
  旧地址：
      11010
      01111
  &   01010  寻址得到地址10

  新地址
      11010
      11111
  &   11010  寻址得到地址26，为旧地址加上旧表大小

  扩容移动判断
  是否移动
      11010
      10000
  &   10000   得到值不为0，则需要移动到当前地址10 + 旧表大小16 = 26
  ```

