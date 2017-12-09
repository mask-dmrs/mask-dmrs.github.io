---
title: builder
permalink: builder
date: 2017-12-09 12:59:38
categories: 设计模式
tags:
 - Java
 - 中级难度
photos:
 - http://static.zuul.top/java-design-patterns-doc-cn/07_builder.png
---
https://github.com/mask-dmrs/java-design-patterns/tree/my-master/07_builder

## 意图
分离复杂的对象从它的结构
代表性，使相同的施工过程可以创造不同的
表示。

## 说明

真实世界的例子

> 想象一下角色扮演游戏的角色生成器。最简单的选择是让电脑为你创造角色。但是如果你想选择职业，性别，发色等字符细节，字符的生成就成了一个循序渐进的过程，当所有的选择都准备好的时候就完成了。

用简单的话来说

> 允许您创建不同的对象风格，同时避免构造器污染。当可能有几种风格的对象时很有用。或者当创建对象时涉及很多步骤。

维基百科说

> 构建器模式是一种对象创建软件设计模式，旨在寻找伸缩构造器反模式的解决方案。

话虽如此，让我加一些关于什么伸缩构造反模式。在某一点或其他我们都看到了如下构造函数：
```java
public Hero(Profession profession, String name, HairType hairType, HairColor hairColor, Armor armor, Weapon weapon) {
}
```

正如你所看到的，构造函数参数的数量可能会很快失去控制，并且可能很难理解参数的排列。 再加上这个参数列表可以继续增长，如果你想在将来添加更多的选项。 这被称为伸缩构造反模式。

**编程示例**

理智的选择是使用Builder模式。 首先我们有我们想要创造的英雄

```java
public final class Hero {
  private final Profession profession;
  private final String name;
  private final HairType hairType;
  private final HairColor hairColor;
  private final Armor armor;
  private final Weapon weapon;

  private Hero(Builder builder) {
    this.profession = builder.profession;
    this.name = builder.name;
    this.hairColor = builder.hairColor;
    this.hairType = builder.hairType;
    this.weapon = builder.weapon;
    this.armor = builder.armor;
  }
}
```

然后我们有建设者

```java
  public static class Builder {
    private final Profession profession;
    private final String name;
    private HairType hairType;
    private HairColor hairColor;
    private Armor armor;
    private Weapon weapon;

    public Builder(Profession profession, String name) {
      if (profession == null || name == null) {
        throw new IllegalArgumentException("profession and name can not be null");
      }
      this.profession = profession;
      this.name = name;
    }

    public Builder withHairType(HairType hairType) {
      this.hairType = hairType;
      return this;
    }

    public Builder withHairColor(HairColor hairColor) {
      this.hairColor = hairColor;
      return this;
    }

    public Builder withArmor(Armor armor) {
      this.armor = armor;
      return this;
    }

    public Builder withWeapon(Weapon weapon) {
      this.weapon = weapon;
      return this;
    }

    public Hero build() {
      return new Hero(this);
    }
  }
```

然后它可以用作：

```java
Hero mage = new Hero.Builder(Profession.MAGE, "Riobard").withHairColor(HairColor.BLACK).withWeapon(Weapon.DAGGER).build();
```
## 适用性
使用Builder模式时

* 创建复杂对象的算法应该独立于组成对象的部分以及如何组装
* 施工过程必须允许对构建的对象进行不同的表示

## 示例

* [java.lang.StringBuilder](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html)
* [java.nio.ByteBuffer](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-) as well as similar buffers such as FloatBuffer, IntBuffer and so on.
* [java.lang.StringBuffer](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
* All implementations of [java.lang.Appendable](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)
* [Apache Camel builders](https://github.com/apache/camel/tree/0e195428ee04531be27a0b659005e3aa8d159d23/camel-core/src/main/java/org/apache/camel/builder)

## 参考

* [Design Patterns: Elements of Reusable Object-Oriented Software](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
* [Effective Java (2nd Edition)](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683)
