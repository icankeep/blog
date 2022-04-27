List
ArrayList LinkedList
class MyArrayList{

  private Object[] values;
  private int size;

  public void add(int index, Object o) {
    // 判断index边界
    if (index < 0 || index >= values.length) {
      throw new ArrayOutOfIndexException();
    }

    // 扩容判断
    ensure();

    // copy
    System.copyOf(values, index, index + 1, size - index);
    values[index] = o;
  }

  // 判断是否需要扩容
  public void ensure() {
    if (size < values.length) {
      return;
    }

    int newSize = values.length + values.length >> 1;
    // 溢出边界判断
    if (newSize < 0) {
      newSize = Integer.MAX_VALUE - 8;
    }

    Object newValues = new Object[newSize];
    for (int i = 0; i < size; i++) {
      newValues[i] = values[i];
    }

    values = newValues;
  }

  public void del(int index) {
    if (index < 0 || index >= values.length) {
      throw new ArrayOutOfIndexException();
    } 

    for (int i = index; i < size - 1; i++) {
      values[i] = values[i + 1]
    }
    values[size - 1] = null;
  }
}


Map
HashMap TreeMap


class MyHashMap {

  class Node {
    Object key;
    Object val;
    Node next;
  }

  private Node[] values;

  private int size;

  public Object get(Object key) {
    // 生成hash, 获取index
    int index = hash(key);

    // 通过equals判断key
    Node node = values[index];
    while (node != null) {
      if (node.equals(key)) {
        return node.val;
      }
      node = node.next;
    }
    return null;
  }

  private int hash(Object key) {
    if (key == null) return 0;
    return key.hashCode() & (size - 1);
  }

}


gc 垃圾发现方法
栈 堆

复制算法 

锁 
分布式锁 zk
CAS 

Mysql 索引



N个无序整数  求最小的K个；

public int[] small(int[] nums,int k) {
  if (nums == null || k <= 0) return new int[0];
  PriorityQueue<Integer> queue = new PriorityQueue<>({
    public int compare(Integer o1, Integer o2) {
      return o2.compareTo(o1);
    }
  }, k);

  for (int i = 0; i < nums.length; i++) {
    queue.add(nums[i]);
  } 

  int[] ret = new int[k];
  for (int i = 0; i < k; i++) {
    ret[i] = queue.poll();
  }
  return ret;
}




给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
 

示例 1：

输入：s = "()"
输出：true
示例 2：

输入：s = "()[]{}"
输出：true
示例 3：

输入：s = "(]"
输出：false
示例 4：

输入：s = "([)]"
输出：false
示例 5：

输入：s = "{[]}"
输出：true
 

提示：

1 <= s.length <= 104
s 仅由括号 '()[]{}' 组成

栈 O N;

典型秒杀系统设计
--库存超卖
--客户端限流的降级 服务端的降级
--突发流量扩容
