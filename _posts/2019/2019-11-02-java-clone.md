---
layout: post
title: Java浅拷贝和深拷贝
category: java
keywords: clone
no-post-nav: true
---

## 什么是浅拷贝和深拷贝
## 浅拷贝
浅拷贝是指我们拷贝出来的对象内部的引用类型变量和原来对象内部引用类型变量是同一引用（指向同一一块内存地址），即修改拷贝对象内部引用类型成员变量值会同步修改原对象内部对应引用类型成员变量的值。   
简单来说，新（拷贝产生）、旧（原对象）对象不同，但是内部如果有引用类型的变量，新、旧对象引用类型属性的引用指向的都是同一块内存地址。

## 深拷贝
深拷贝是指全部拷贝原对象的内容，包括内部的引用类型也进行拷贝。新、旧对象完全没有联系。

## Object类clone方法是浅拷贝还是深拷贝
我们都知道Object类的clone方法是一个native方法，该方法通过protected修饰，必须得通过子类重写该方法，并且调整访问修饰符为public方能通过实例对象在需要拷贝对象的地方使用。除此之外，重写clone方法的类必须实现Cloneable接口，否则将会抛出CloneNotSupportedException异常。通过下面的代码测试Object类clone方法是什么拷贝。
```java
public class Student implements Cloneable {

    /**
     * 年龄
     */
    private Integer age;
    /**
     * 姓名
     */
    private String name;
    /**
     * 分数
     */
    private int[] scores;

    @Override
    public Student clone() throws CloneNotSupportedException {
        return (Student) super.clone();
    }

    @Override
    public String toString() {
        return String.format("age:%d, name:%s, scores:%s", age, name, Arrays.toString(scores));
    }

    public Student(Integer age, String name, int[] scores) {
        this.age = age;
        this.name = name;
        this.scores = scores;
    }

    public void change(){
        if (age != null){
            age++;
        }
        if (scores.length > 0){
            scores[0]++;
        }
        name = "changed";
    }

}

public class Test {

    public static void main(String[] args) throws Exception {
        Student student = new Student(20, "wyc1856", new int[]{59, 80, 70});
        System.out.println(student);
        //通过Object.clone()方法进行clone
        Student clone = student.clone();
        //修改克隆的对象属性
        clone.change();
        //打印原始student对象
        System.out.println(student);
    }
    
}
```

输出结果:
```
age:20, name:wyc1856, scores:[59, 80, 70]
age:20, name:wyc1856, scores:[60, 80, 70]
```

通过输出结果scores的第一个元素从59->60可以看出来Object类的clone方法是浅拷贝，如果你debug也会发现克隆对象clone内的age和scores的引用和原对象student内对应属性的引用相同，debug的截图如下：
![2019-12-02-21-31-30](http://image.wyc1856.club/2019-12-02-21-31-30.png)
你可能就有疑问了，change方法也更新了age和name，为啥它俩却没变呢？因为age是Integer类型，在进行age++操作的时候先拆箱，再进行+1操作，最后进行装箱的时候会重新生成一个Integer对象。至于name为啥没变是因为String类型是不可变类，每次对String对象的修改都会指向一个新的内存空间，至于String为啥设计成不可变的，可以看看[这篇文章](https://juejin.im/post/59cef72b518825276f49fe40)。

## 如何实现深拷贝
## 1.手动实现clone逻辑
通过实现cloneable接口，重写clone方法，对引用类型成员变量进行深拷贝，从而实现整个对象的深拷贝。例如刚才的Student例子，重写clone方法：
```java
@Override
public Student clone() throws CloneNotSupportedException {
    Student student = (Student) super.clone();
    int[] scores = new int[this.scores.length];
    System.arraycopy(this.scores, 0, scores, 0, scores.length);
    student.scores = scores;
    return student;
}
```

### 2.通过java原生序列化和反序列化
我们可以通过将要克隆的类实现Serializable接口，使其具备序列化和反序列化的能力，同时，其内部的成员变量也要实现序列化接口才行。clone的过程就是将实例对象先序列化为二进制字节流，然后再反序列化为一个全新的克隆对象，代码实现：
```java
public static <T extends Serializable> T clone(T obj) {
        T cloneObj = null;
        try {
            //写入字节流
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(obj);
            oos.close();

            //分配内存,写入原始对象,生成新对象
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());//获取上面的输出字节流
            ObjectInputStream ois = new ObjectInputStream(bais);

            //返回生成的新对象
            cloneObj = (T) ois.readObject();
            ois.close();
            baos.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return cloneObj;
    }
```

### 3.通过json序列化和反序列化
其实思想还是先进行序列化，然后在进行反序列化生产新对象，只是序列化方式变成json字符串了。可依赖第三方json序列化类库实现，例如jackson、gson、fastjson等。