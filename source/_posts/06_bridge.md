---
title: bridge
permalink: bridge
date: 2017-12-09 12:59:24
categories: 设计模式
tags:
 - Java
 - 中级难度
photos:
 - https://static.zuul.top/java-design-patterns-doc-cn/06_bridge.png
---
https://github.com/mask-dmrs/java-design-patterns/tree/my-master/06_bridge
## 也被称为
手柄/身体

## 意图
将一个抽象与其实现分离开来，以使两者可以独立地变化。

## 说明

真实世界的例子

> 考虑你有一个不同的附魔武器，你应该允许混合不同的武器和不同的附魔。你会怎么做？为每个附魔创建每个武器的多个副本，或者你会创建单独的附魔，并根据需要将其设置为武器？桥梁模式可以让你做第二个。

用普通话说

桥梁模式是关于比继承更喜欢构图。实现细节从一个层次结构推送到另一个具有单独层次结构的对象。

维基百科说

> 桥梁模式是软件工程中使用的一种设计模式，旨在“将抽象与其实现分离开，以便二者可以独立地变化”

**编程示例**

从上面翻译我们的武器示例。这里我们有`Weapon`层次结构

```java
public interface Weapon {
  void wield();
  void swing();
  void unwield();
  Enchantment getEnchantment();
}

public class Sword implements Weapon {

  private final Enchantment enchantment;

  public Sword(Enchantment enchantment) {
    this.enchantment = enchantment;
  }

  @Override
  public void wield() {
    LOGGER.info("The sword is wielded.");
    enchantment.onActivate();
  }

  @Override
  public void swing() {
    LOGGER.info("The sword is swinged.");
    enchantment.apply();
  }

  @Override
  public void unwield() {
    LOGGER.info("The sword is unwielded.");
    enchantment.onDeactivate();
  }

  @Override
  public Enchantment getEnchantment() {
    return enchantment;
  }
}

public class Hammer implements Weapon {

  private final Enchantment enchantment;

  public Hammer(Enchantment enchantment) {
    this.enchantment = enchantment;
  }

  @Override
  public void wield() {
    LOGGER.info("The hammer is wielded.");
    enchantment.onActivate();
  }

  @Override
  public void swing() {
    LOGGER.info("The hammer is swinged.");
    enchantment.apply();
  }

  @Override
  public void unwield() {
    LOGGER.info("The hammer is unwielded.");
    enchantment.onDeactivate();
  }

  @Override
  public Enchantment getEnchantment() {
    return enchantment;
  }
}
```

和单独的结界层次

```java
public interface Enchantment {
  void onActivate();
  void apply();
  void onDeactivate();
}

public class FlyingEnchantment implements Enchantment {

  @Override
  public void onActivate() {
    LOGGER.info("The item begins to glow faintly.");
  }

  @Override
  public void apply() {
    LOGGER.info("The item flies and strikes the enemies finally returning to owner's hand.");
  }

  @Override
  public void onDeactivate() {
    LOGGER.info("The item's glow fades.");
  }
}

public class SoulEatingEnchantment implements Enchantment {

  @Override
  public void onActivate() {
    LOGGER.info("The item spreads bloodlust.");
  }

  @Override
  public void apply() {
    LOGGER.info("The item eats the soul of enemies.");
  }

  @Override
  public void onDeactivate() {
    LOGGER.info("Bloodlust slowly disappears.");
  }
}
```

而且这两个行动层次

```java
Sword enchantedSword = new Sword(new SoulEatingEnchantment());
enchantedSword.wield();
enchantedSword.swing();
enchantedSword.unwield();
// The sword is wielded.
// The item spreads bloodlust.
// The sword is swinged.
// The item eats the soul of enemies.
// The sword is unwielded.
// Bloodlust slowly disappears.

Hammer hammer = new Hammer(new FlyingEnchantment());
hammer.wield();
hammer.swing();
hammer.unwield();
// The hammer is wielded.
// The item begins to glow faintly.
// The hammer is swinged.
// The item flies and strikes the enemies finally returning to owner's hand.
// The hammer is unwielded.
// The item's glow fades.
```

## 适用性
使用桥梁模式时

* 您想避免抽象与其实现之间的永久绑定。例如，当实现必须在运行时被选择或切换时，情况可能如此。
* 抽象和它们的实现都应该通过子类来扩展。在这种情况下，Bridge模式可以让您将不同的抽象和实现组合起来，并将其独立扩展
* 抽象的实施变化不应该对客户产生影响;也就是说，他们的代码不应该被重新编译。
* 你有一个类扩散。这样的类层次结构表示需要将对象分成两部分。 Rumbaugh使用术语“嵌套概括”来指代这样的类层次结构
* 您想要在多个对象之间共享一个实现（可能使用引用计数），这个事实应该隐藏在客户端。一个简单的例子是Coplien的String类，其中多个对象可以共享相同的字符串表示。

## 参考

* [Design Patterns: Elements of Reusable Object-Oriented Software](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
