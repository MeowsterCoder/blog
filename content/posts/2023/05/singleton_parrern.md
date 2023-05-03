+++
title = "单例模式"
date = "2023-05-04T01:19:51+08:00"
author = "Black"
authorTwitter = "" #do not include @
cover = ""
tags = ["Java", "Design Pattern"]
categories = ["Java Design Pattern"]
keywords = ["Java", "Design Pattern", "Singleton Pattern"]
description = "单例模式的实现: 饿汉式、懒汉式、枚举式"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

单例模式是指在任何时候，都只有一个实例，比如DataSource

## 饿汉式单例
饿汉式单例是指在类创建时就对实例进行创建，不需要锁，所以效率高
{{< code language="java" title="饿汉式单例" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
public class HungrySingleton {
    private static final HungrySingleton hungrySingleton= new HungrySingleton();

    private HungrySingleton() {
    }

    public static HungrySingleton getInstance() {
        return hungrySingleton;
    }
}
{{</ code >}}

## 懒汉式单例  
懒汉式单例是指在使用时再对实例进行创建，但是在多线程情况下会出现重复创建实例的情况，所以需要双重检查锁对其进行检查  同时使用volatile确保内存可见性   
相比饿汉式单例，懒汉式单例在不使用时，就不会创建对象，节约了内存开销，所以普遍认为懒汉式效率比较高  
{{< code language="java" title="懒汉式单例" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
public class LazySingleton {
    public volatile static LazySingleton lazySingleton;

    private LazySingleton() {
    }

    public static LazySingleton getInstance() {
        if (lazySingleton == null) {
            synchronized (LazySingleton.class) {
                if (lazySingleton == null) {
                    lazySingleton = new LazySingleton();
                }
            }
        }
        return lazySingleton;
    }
}

{{</ code >}}

## 枚举式单例
在Effective Java中提到了一种利用Enum实现单例模式  
> 第3条: 用私有构造器和或者枚举类型强化singleton属性  
> 枚举单例实现更加简介，无偿提供了序列化机制，即时是复杂的序列化和反射攻击的时候，也可以防止多次实例化。


{{< code language="java" title="枚举式的单例" id="3" expand="Show" collapse="Hide" isCollapsed="false" >}}
public enum EnumSingleton {
    INSTANCE;

    private Object data;

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public static EnumSingleton getInstance() {
        return INSTANCE;
    }
}
{{</ code >}}