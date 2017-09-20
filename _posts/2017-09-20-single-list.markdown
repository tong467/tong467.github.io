---
layout: post
title: Conference on Javascript
date: 2017-09-20 14:00:00 +0800
description: 单向链表倒叙，及特定节点O(1)时间删除。
img: single-list-1.jpg
tags: [单向链表, 面试,java 基础]
---
最近多家公司面试都被问道了关于单项链表的问题，主要集中在倒叙，及其删除特定节点，即给你待删除的节点，让你删除它。为了加深印象特此记录。


构建链表节点，包含一个它存储的对象，及其下一个节点的对象；
{% highlight java %}
private class Node<E> {

    private E item;
    private Node<E> next;

    public Node(E element, Node<E> next) {
        this.item = element;
        this.next = next;
    }

    public E getItem() {
        return item;
    }

    public void setItem(E item) {
        this.item = item;
    }

    public Node<E> getNext() {
        return next;
    }

    public void setNext(Node<E> next) {
        this.next = next;
    }
    
}
{% endhighlight %}
构建链表对象 包含一个头结点，及一个长度属性，
{% highlight java %}
public class SingleList<E> {

    private Node<E> head;

    private int size;

    public SingleList() {

    }

    public SingleList(E element) {
        head = new Node(element, null);
        size = 1;
    }

    /**
     * 链表末尾增加一个节点
     *
     * @param element 节点的内容
     */
    public SingleList add(E element) {
        Node item = new Node(element, null);
        getLast().setNext(item);
        size++;
        return this;
    }

    /**
     * 获取最后一个节点
     */
    private Node<E> getLast() {
        return getNode(size - 1);
    }

    /**
     * 获取特定index 节点
     *
     * @param index 节点下标
     * @return 节点
     */
    public Node<E> getNode(int index) {
        Node<E> node = head;
        if (size == 0 || index >= size || index < 0) {
            return node;
        }
        for (int i = 0; i < size; i++) {
            if (i != index) {
                node = node.getNext();
            } else {
                break;
            }
        }
        return node;
    }
}
{% endhighlight %}
下面说下倒叙的方法思路，目前往上主流2种解决方案，循环，递归。
首选咱们说下循环，循环的思路就是将链表循环一遍，每循环到1个节点，把这个节点从原有链表脱离，设置为新链表的表头。代码如下：
{% highlight java %}
    	public void reverseList() {
        //上一个节点用于存储倒叙的head
        Node<E> prev = null;
        //当前节点，用于脱离原链表建立新链表
        Node<E> node = head;
        //原链表引用用于分离节点
        Node<E> next = head;
        while (next != null) {
            //原链表引用，引用向后移动一位，保证接下来操作不会影响
            next = next.getNext();
            //脱离原链表加入新链表
            node.setNext(prev);
            //更新新链表head
            prev = node;
            //准备下一次循环要加入新链表的节点
            node = next;
        }
        head = prev;
    }
{% endhighlight %}
再来咱说说下递归的思路，递归，其实算是把所有的节点入栈，然后在出栈的时候把所有的引用反转。 代码如下：
{% highlight java %}
    public void reverseList1() {
        head = reverseList1(head);
    }

    private Node<E> reverseList1(Node<E> head) {
        if (head == null || head.getNext() == null) {
            return head;
        } else {
            //为了返原有的最后一个节点反转后将变为head节点
            Node<E> reversedHead = reverseList1(head.getNext());
            // 引用反转将自己的next节点的next 节点指向自己
            head.getNext().setNext(head);
            // 破坏以前自己指向下一个节点 设置为null
            head.setNext(null);
            // 层层传递给最上面的
            return reversedHead;
        }
    }
{% endhighlight %}
下面我们来写下删除特定的节点，对于单项链表来说，节点的重要意义是他是他上一个节点的下一个节点，和他说他下一个节点的上一个节点。顺着这个思路想，其实删除这个节点，最主要的就是让他的上一个节点指向他的下一个节点，他就不存在。所以其实我们要做的就是把他的下个节点的值赋给他把他的next指向next的next。但是有个特例如果你要删除的是末尾节点就不能用这个方法，因为末尾节点没有下一个节点，也就是说他存在的意义就是它是它上一个节点的下一个节点，也即是说他自己要变为null 因为java是引用也就是说你把自己赋值为null也没用，只能把自己的上一个节点的next界定啊 代码如下：
{% highlight java %}
    public E removie(Node<E> node) {
        E item = null;
        if (node.getNext() == null) {
            item = removie(size - 1);
        } else {
            item = node.getItem();
            node.setItem(node.getNext().getItem());
            node.setNext(node.getNext().getNext());
        }
        return item;
    }
    /**
     * 删除特定index 节点
     *
     * @param index 节点下标
     * @return 节点内容
     */
    public E removie(int index) {
        E item = null;
        Node<E> node = head;
        if (size == 0 || index >= size || index < 0) {
            return item;
        }
        for (int i = 0; i < size; i++) {
            if (i != index - 1) {
                node = node.getNext();
            } else {
                item = node.getNext().getItem();
                node.setNext(node.getNext().getNext());
            }

        }
        return item;
    }
{% endhighlight %}