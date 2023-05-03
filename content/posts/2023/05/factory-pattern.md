+++
title = "工厂模式"
date = "2023-05-02T23:30:10+08:00"
author = "Black"
authorTwitter = "" #do not include @
cover = ""
tags = ["Java", "Design Pattern"]
categories = ["Java Design Pattern"]
keywords = ["Java", "Design Pattern", "Factory Pattern"]
description = "工厂模式的实现方法，简单工厂模式、工厂方法模式、抽象工厂模式"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++


## 简单工厂模式
> 感觉我之前学的有点老了，最新的可以去参考[Factory](https://java-design-patterns.com/zh/patterns/factory/)

简单工厂模式，属于创建型模式，不属于GOF23种设计模式之一。  
主要适用于生成对象较少的场景，不需要关心类的创建逻辑。  
缺点是违背了开闭原则，增加新的类需要去修改工厂类的判断逻辑，不易于扩展。

示例：
这里拿糖果工厂举例
{{< code language="java" title="创建统一的接口" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
// 创建糖果接口，调用make方法去制造糖果
public interface Candy {
    void make();
}
{{</ code >}}
{{< code language="java" title="创建接口实现类" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
// 创建星形糖果类，实现Candy接口，重写make方法
public class StarCandy implements Candy {
    @Override
    public void make() {
        System.out.println("制作了星形糖果");
    }
}


// 创建爱心糖果类，实现Candy接口，重写make方法
public class LoveCandy implements Candy {
    @Override
    public void make() {
        System.out.println("制作了爱心糖果");
    }
}
{{</ code >}}

{{< code language="java" title="创建简单工厂模式" id="3" expand="Show" collapse="Hide" isCollapsed="false" >}}
//创建糖果工厂
public class CandyFactory {
    public Candy create(Class<? extends Candy> clazz) {
        try {
            if (clazz != null) {
                return (Candy)clazz.newInstance();
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
{{</ code >}}

{{< code language="java" title="调用" id="4" expand="Show" collapse="Hide" isCollapsed="false" >}}
public class Main {
    public static void main(String[] args) {
        Candy candy = new CandyFactory().create(LoveCandy.class);
        candy.make();
        // 执行后就会打印 制作了爱心糖果
    }
}
{{</ code >}}

{{< figure src="/image/posts/2023-05-03_00-28.png" alt="类图" position="center" style="border-radius: 8px;" caption="类图" captionPosition="center" captionStyle="color: black;" >}}

---

## 工厂方法模式

> 为创建一个对象定义一个接口，但是让子类决定实例化哪个类。工厂方法允许类将实例化延迟到子类。  
> 可以参考[Factory Method](https://java-design-patterns.com/zh/patterns/factory-method/)

优点是可以不用关心类的创建细节，符合开闭原则，提高了扩展性  
缺点是类的数量容易增加，增加了代码结构的复杂程度，增加了系统的抽象性和理解难度

{{< code language="java" title="创建接口并进行实现" id="5" expand="Show" collapse="Hide" isCollapsed="false" >}}
// 创建糖果接口，调用make方法去制造糖果
public interface Candy {
    void make();
}

public class StarCandy implements Candy {
    @Override
    public void make() {
        System.out.println("制作了星形糖果");
    }
}

public class LoveCandy implements Candy {
    @Override
    public void make() {
        System.out.println("制作了爱心糖果");
    }
}
{{</ code >}}

{{< code language="java" title="创建工厂接口并进行实现" id="6" expand="Show" collapse="Hide" isCollapsed="false" >}}
public interface CandyFactory {
    Candy create();
}

public class LoveCandyFactory implements CandyFactory{
    @Override
    public Candy create() {
        return new LoveCandy();
    }
}

public class StarCandyFactory implements CandyFactory{
    @Override
    public Candy create() {
        return new StarCandy();
    }
}

{{</ code >}}

{{< code language="java" title="调用" id="7" expand="Show" collapse="Hide" isCollapsed="false" >}}
public class Main {
    public static void main(String[] args) {
        CandyFactory factory = new LoveCandyFactory();
        factory.create().make();
    }
}
{{</ code >}}

{{< figure src="/image/posts/2023-05-03_00-50.png" alt="类图" position="center" style="border-radius: 8px;" caption="类图" captionPosition="center" captionStyle="color: black;" >}}

---

## 抽象工厂
> 创建型模式，提供一个可以创建一个系列或者互相依赖的对象的接口，不需要指定具体的类。  
> 可以参考[Abstract Factory](https://java-design-patterns.com/zh/patterns/abstract-factory/)

利用抽象工厂，我们可以直接批量创建一系列的对象，不需要关心对象的实现细节  
缺点是对其进行更新时我们需要去修改对应的细节  

这里我们采用java-design-patterns中类似的参考例子  
首先每个王国都有国王和军队，国王和军队有各自的行为，我们先分别创建人类和恶魔的国王和军队，并分别实现自己的对应的行为  
{{< code language="java" title="创建抽象类并进行实现" id="8" expand="Show" collapse="Hide" isCollapsed="false" >}}
public interface King {
    void order();
}

public interface Army {
    void attack();
}

public class HumanKing implements King{
    @Override
    public void order() {
        System.out.println("命令人类军队出击！");
    }
}

public class HumanArmy implements Army{
    @Override
    public void attack() {
        System.out.println("人类军队出击！");
    }
}

public class DemonKing implements King{
    @Override
    public void order() {
        System.out.println("命令魔族军队出击！");
    }
}

public class DemonArmy implements Army{
    @Override
    public void attack() {
        System.out.println("魔族军队出击！");
    }
}

{{</ code >}}

然后我们在创建王国工厂，用于创建一个王国
{{< code language="java" title="创建工厂方法并进行实现" id="9" expand="Show" collapse="Hide" isCollapsed="false" >}}
//这里采用的是接口，如果有公共逻辑，则可以使用抽象类
public interface KingdomFactory {
    King createKing();
    Army createArmy();
}

public class HumanKingdomFactory implements KingdomFactory{
    @Override
    public King createKing() {
        return new HumanKing();
    }

    @Override
    public Army createArmy() {
        return new HumanArmy();
    }
}

public class DemonKingdomFactory implements KingdomFactory{
    @Override
    public King createKing() {
        return new DemonKing();
    }

    @Override
    public Army createArmy() {
        return new DemonArmy();
    }
}
{{</ code >}}

{{< code language="java" title="调用" id="10" expand="Show" collapse="Hide" isCollapsed="false" >}}
public class Main {
    public static void main(String[] args) {
        KingdomFactory humanKindom = new HumanKingdomFactory();
        humanKindom.createKing().order();
        humanKindom.createArmy().attack();
    }
}
{{</ code >}}
{{< figure src="/image/posts/2023-05-04_00-20.png" alt="类图" position="center" style="border-radius: 8px;" caption="类图" captionPosition="center" captionStyle="color: black;" >}}