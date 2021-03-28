---
layout: default
title: java单向链表、双向链表、环链表
#excerpt: 
---

　　链表是程序里重要的数据结构，在程序世界运用很广泛，众所周知的当属于jdk里的linkList了。  

　　链表的优点，是相对于数组来说，扩容是非常快的，如果是数组扩容，数组是新申请一个更大空间的数组，然后把老数组内的数据复制到新数组；而链表就不必申请新链表，直接再分配一个元素的存储空间即可。  

　　链表的缺点，相对于数组来说，数组的访问元素是直接根据下标来的，时间复杂度是O(1)；而链表则需要遍历整个链表，最差的情况，时间复杂度是O(N)。

# 单向链表

```java
public class SinglerNodeTest {
    private Node head;

    public static void main(String[] args) {
        SinglerNodeTest sn = new SinglerNodeTest();

        sn.add("1");
        sn.add("2");
        sn.add("3");

        Node n = sn.get("1");
        System.out.println(n);

        sn.remove("2");

        sn.show();
    }

    private void add(String data) {
        if (null == head) {
            head = new Node();
            head.data = data;
            return;
        }

        Node tmp = head;
        while (null != tmp.next) {
            tmp = tmp.next;
        }

        Node n2 = new Node();
        n2.data = data;
        tmp.next = n2;
    }

    private void remove(String data) {
        if (null != head && data.equals(head.data)) {
            if (null == head.next) {
                head = null;
            } else {
                head = head.next;
            }
            return;
        }

        Node tmp = head;
        while (null != tmp.next) {
            if (data.equals(tmp.next.data)) {
                // 跳过tmp.next节点
                tmp.next = tmp.next.next;
                return;
            }
            tmp = tmp.next;
        }
    }

    private Node get(String data) {
        Node tmp = head;
        while (null != tmp) {
            if (data.equals(tmp.data)) {
                return tmp;
            }

            tmp = tmp.next;
        }
        return null;
    }

    private void show() {
        Node tmp = head;
        while (null != tmp) {
            System.out.print(tmp.data + "-->");
            tmp = tmp.next;
        }
    }

    static class Node {
        private String data;
        private Node next;

        @Override
        public String toString() {
            return data;
        }
    }
}
```


# 双向链表

```java
public class DoubleNodeTest {
    private Node head;

    public static void main(String[] args) {
        DoubleNodeTest dn = new DoubleNodeTest();

        dn.add("1");
        dn.add("2");
        dn.add("3");
        dn.add("4");
        dn.add("5");

        Node n = dn.get("5");
//        System.out.println(n);

        dn.show();
        System.out.println();
        dn.remove("5");

        dn.show();
    }

    private void add(String data) {
        if (null == head) {
            head = new Node();
            head.data = data;
            return;
        }

        Node tmp = head;
        while (null != tmp.next) {
            tmp = tmp.next;
        }

        Node n2 = new Node();
        n2.data = data;
        // 把上一个节点，挂到新生成节点的pre
        n2.pre = tmp;

        // 把新生成的节点挂到上一个节点
        tmp.next = n2;
    }

    private Node get(String data) {
        Node tmp = head;
        while (null != tmp) {
            if (data.equals(tmp.data)) {
                return tmp;
            }

            tmp = tmp.next;
        }
        return null;
    }

    private void remove(String data) {
        if (null != head && head.data.equals(data)) {
            if (null == head.next) {
                head = null;
            } else {
                head = head.next;
                head.pre = null;
            }
            return;
        }

        Node tmp = head;
        while (null != tmp.next) {
            if (tmp.next.data.equals(data)) {
                // 下两位节点
                Node next2node = tmp.next.next;
                // 节点前移一位
                tmp.next = next2node;
                // 下两位节点的pre节点，前移
                if (null != next2node) {
                    next2node.pre = tmp;
                }
                return;
            }
            tmp = tmp.next;
        }
    }

    private void show() {
        if (null == head) {
            System.out.println();
        } else {
            Node tmp = head;
            while (null != tmp) {
                System.out.print("(" + tmp.pre + "," + tmp + "," + tmp.next + "),");
                tmp = tmp.next;
            }
        }
    }

    static class Node {
        private String data;
        private Node pre;
        private Node next;

        @Override
        public String toString() {
            return data;
        }
    }
}
```


# 环链表

```java
public class CycleNodeTest {
    private Node head;
    private int count = 0;

    public static void main(String[] args) {
        CycleNodeTest dn = new CycleNodeTest();

        dn.add("1");
        dn.add("2");
        dn.add("3");
        dn.add("4");
        dn.add("5");

        Node n = dn.get("5");
//        System.out.println(n);

        dn.show();
        dn.remove("5");

        System.out.println();
        dn.show();
    }

    private void add(String data) {
        if (null == head) {
            head = new Node();
            head.data = data;

            // 計數器+1
            count++;
            return;
        }

        Node tmp = head;
        for (int i = 1; i < count; i++) {
            tmp = tmp.next;
        }

        Node n2 = new Node();
        n2.data = data;
        // 把上一个节点，挂到新生成节点的pre
        n2.pre = tmp;

        // 把新生成的节点挂到上一个节点
        tmp.next = n2;

        /**
         * 闭环操作
         */
        // 首节点的pre是新生成的节点
        head.pre = n2;
        // 新生成的节点的next是首节点
        n2.next = head;

        // 計數器+1
        count++;
    }

    private void remove(String data) {
        if (null == head) {
            return;
        }

        Node tmp = head;
        for (int i = 0; i < count; i++) {
            if (tmp.data.equals(data)) {
                Node pre = tmp.pre;
                Node next = tmp.next;

                pre.next = next;
                next.pre = pre;

                if (i == 0) {
                    // 当删除的是首节点的时候(因为首节点比较特殊，它是暴露给外面调用的，若把首节点的pre和next指向null，调用show()的时候就空指针的了，所以，只有覆盖首节点)
                    head = next;
                } else {
                    // 中间节点
                    tmp.next = null;
                    tmp.pre = null;
                }

                count--;
                return;
            }
            tmp = tmp.next;
        }
    }

    private Node get(String data) {
        Node tmp = head;
        for (int i = 0; i < count; i++) {
            if (data.equals(tmp.data)) {
                return tmp;
            }

            tmp = tmp.next;
        }
        return null;
    }

    private void show() {
        Node tmp = head;
        for (int i = 0; i < count; i++) {
            System.out.print("(" + tmp.pre + "," + tmp + "," + tmp.next + "),");
            tmp = tmp.next;
        }
    }

    static class Node {
        private String data;
        private Node pre;
        private Node next;

        @Override
        public String toString() {
            return data;
        }
    }

}
```

