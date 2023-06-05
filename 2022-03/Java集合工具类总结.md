## List
### ArrayList
底层是数组，数组的特性都有，按照index随机访问时间复杂度O(1)
增：O(1) 需要考虑扩容，每次扩容1.5倍
删：O(n) 被删除元素后面的需要统一往前移
改：O(1) 
查：O(1)

### LinkedList
底层是链表
增：O(1) 
删：O(n) 需要遍历链表
改：O(n) 需要遍历链表
查：O(n)

## Map/Set
### HashMap/HashSet

### ConcurrentHashMap

### LinkedHashMap/LinkedHashSet

### TreeMap/TreeSet
底层基于红黑树

### Hashtable

## Queue
### PriorityQueue

### DelayQueue

### ArrayBlockingQueue
基于ReentrantLock实现阻塞等待以及唤醒

### ConcurrentLinkedQueue

### 

## Stack
底层继承的是Vector

数据存储一样是数组

