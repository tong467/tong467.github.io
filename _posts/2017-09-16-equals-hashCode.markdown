---
layout: post
title: "equals and hashCode"
date: 2017-09-15 19:00:20 +0800
description: 一次面试经历让我对equals and hashCode的回顾。
img:  # Add image post (optional)
tags: [java 基础, 面试 ] # add tag
---
一次面试经历让我对equals 和 hashCode的回顾。

* equals 方法定义  public boolean equals(Object object) {}
* 入参与本身对象内存地址是否一致
* 入参为空情况
* 入参的类型是否一致
* 本身属性为空的情况。
如果你想把相同的对象放入HashSet只保留一个要重写equals和hashCode 方法。如果只重写hashCode方法依然会当两个对象处理
如果你重写了hashCode方法，放入HashSet后不要修改参与计算的属性，否者会导致不可预知问题。

昨天面试官问我equals方法是做什么用的，有没有什么情况下是需要必须重写equals方法的。并重写一个类的equals方法。

当时我的回答是，让你想判断两个对象的值是否相同而不是比较内存地址时就要重写equals方法。我觉得没有什么情况下是必须重写equals方法的，因为你的equals方法判断逻辑放在调用的地方，一样能达到效果。

当时现场的重写的代码是这样的。
{% highlight java %}
class Person {

    private int age;
    private String name;

    public Person(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Person person) {
        if(this.name.equals(person.name)&&this.age.equals(person.age)){
            return true
        }else{
            return false;
        }
    }
}
{% endhighlight %}
面试官又问空指针考虑了吗？在想想其他的，我说哦，忘了，修改后
{% highlight java %}
 @Override
    public boolean equals(Person person) {
        if(person==null){
            return false;
        }
        if(this.name.equals(person.name)&&this.age.equals(person.age)){
            return true
        }else{
            return false;
        }
    }
{% endhighlight %}
回来查过资料后为我当时回答感到羞愧。回来修改后代码
   
{% highlight java %}

class Person {

    private int age;
    private String name;

    public Person(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object object) {
        if (this == object) {
            return true;
        }
        if (object == null) {
            return false;
        }
        if (getClass() != object.getClass()) {
            return false;
        }
        Person hae = (Person) object;
        if (this.age != hae.age) {
            return false;
        }
        if (name == null) {
            if (hae.name != null) {
                return false;
            } else {
                return true;
            }
        } else {
            return this.name.equals(hae.name);
        }
    }

    @Override
    public int hashCode() {
        return 31 * age + (name != null ? name.hashCode() : 0);
    }

    @Override
    public String toString() {
        return String.format("age:%d name:%s hashCode:%d", age, name, hashCode());
    }
}
{% endhighlight %}
测试代码
{% highlight java %}
public static void main(String[] args) {
        Person p1 = new Person(10, "佟1");
        Person p2 = new Person(10, "佟1");
        System.out.println("p1==p2 :" + p1.equals(p2));
        List<Person> list = new ArrayList<>();
        list.add(p1);
        list.add(p2);
        System.out.println("list size :" + list.size());

        HashMap<Person, Person> hashMap = new HashMap<>();
        hashMap.put(p1, p1);
        hashMap.put(p2, p2);
        System.out.println("hashMap size :" + hashMap.size());
    }

    private static void printMap(HashMap<Person, Person> hashMap) {
        for (Person p : hashMap.keySet()) {
            System.out.print("printMap :");
            System.out.print(p);
            System.out.print("  ");
            System.out.print(hashMap.get(p));
            System.out.println();
        }
    }
{% endhighlight %}
第一次测试结果:

>p1==p2 :true
>
>list size :2
>
>hashMap size :1

重写hashCode和equals方法后以对象为key放入两个p1,p2hashMap只会存在一条记录
如果我修改P2的值，你会发现，现在hashMap中存放的value是P2，key是P1。
{% highlight java %}
......
        hashMap.put(p2, p2);
        p2.setAge(12);
......
{% endhighlight %}
运行结果
>printMap :age:10 name:佟1 hashCode:630248  age:12 name:佟1 hashCode:630310

看值能很清晰的看出key为p1 值为p2
还有一个很好玩的事是，如果你创建一个新的对象p3他的值跟p1一样，你可以以此为key删除HashMap中的值。代码如下
{% highlight java %}
 public static void main(String[] args) {
        Person p1 = new Person(10, "佟1");
        HashMap<Person, Person> hashMap = new HashMap<>();
        hashMap.put(p1, p1);
        Person p3 = new Person(10, "佟1");
        hashMap.remove(p3)
        System.out.println("hashMap size :" + hashMap.size());
    }
{% endhighlight %}

输出结果：
>hashMap size :0

现在我们来做一个更神奇的实验，如果我变更了key也就是p1的值
{% highlight java %}
 public static void main(String[] args) {
        Person p1 = new Person(10, "佟1");
        HashMap<Person, Person> hashMap = new HashMap<>();
        hashMap.put(p1, p1);
        p1.setAge(12);
        printMap(hashMap);
        System.out.println("hashMap size :" + hashMap.size());
        
    }
{% endhighlight %}
输出结果：
> printMap :age:12 name:佟1 hashCode:630310  null
> 
> hashMap size :1

你会发下key的值变了，但是通过key在map中获取获取的结果是null，map的size是1 那是不是说map的值没了呢？现在的map里面存储的就是一个 p1 -> null 的对象呢，我来修改下输出验证下我的想法；
{% highlight java %}
 public static void main(String[] args) {
        Person p1 = new Person(10, "佟1");
        HashMap<Person, Person> hashMap = new HashMap<>();
        hashMap.put(p1, p1);
        p1.setAge(12);
        printMap(hashMap);
        for (Person p : hashMap.values()) {
            System.out.println("value:" + p);
        }
        System.out.println("hashMap size :" + hashMap.size());
    }
{% endhighlight %}
输出结果：
>printMap :age:12 name:佟1 hashCode:630310  null
>
>value:age:12 name:佟1 hashCode:630310
>
>hashMap size :1

这个结果就很玄幻了，map size为1，key 是有值的，value 也是有值的，但是你就是不能通过key获取这个值。并且你也不能通过key删除这个值。但是你换另一方式使用  hashMap.entrySet().iterator(); 然后通过Iterator进行获取key和value 可以正常获取到key，value 和删除。关于这个修改了hashCode具体的值导致的问题，之后之后在研究了。
