# final关键字的作⽤是什么？ 

> 修饰类：表示类不可被继承 

> 修饰⽅法：表示⽅法不可被⼦类覆盖，但是可以重载 

> 修饰变量：表示变量⼀旦被赋值就不可以更改它的值。 

> 修饰成员变量： 

- 如果final修饰的是类变量，只能在静态初始化块中指定初始值或者声明该类变量时指定初始值。 

- 如果final修饰的是成员变量，可以在⾮静态初始化块、声明该变量或者构造器中执⾏初始值。 

> 修饰局部变量： 

- 系统不会为局部变量进⾏初始化，局部变量必须由程序员显示初始化。
- 因此使⽤final修饰局部变量时， 即可以在定义时指定默认值（后⾯的代码不能对变量再赋值），也可以不指定默认值，⽽在后⾯的代码 中对final变量赋初值（仅⼀次） 

```java
class FinalVar {
    final static int a = 0;//再声明的时候就需要赋值 或者静态代码块赋值

    final int b = 0;//再声明的时候就需要赋值  或者代码块中赋值	或者构造器赋值
    
    public static void main(String[] args) {
        final int localA;    //局部变量只声明没有初始化，不会报错,与final⽆关。
        localA = 0;//在使⽤之前⼀定要赋值
        //localA = 1; 但是不允许第⼆次赋值
    }
}
```

> 修饰基本类型数据和引⽤类型数据： 

- 如果是基本数据类型的变量，则其数值⼀旦在初始化之后便不能更改； 

- 如果是引⽤类型的变量，则在对其初始化之后便不能再让其指向另⼀个对象。但是引⽤的值是可变 的。

```java
public static void main() {
    final int[] iArr = {1, 2, 3, 4};
    iArr[2] = -3;//合法
    iArr = null;//⾮法，对iArr不能重新赋值

    final Person p = new Person(25);
    p.setAge(24);//合法
    p = null;//⾮法
}
```

**为什么局部内部类和匿名内部类只能访问局部final变量？**

> ⾸先需要知道的⼀点是: 内部类和外部类是处于同⼀个级别(类级别)的，内部类不会因为定义在⽅法中就会随着⽅法的执⾏完毕就被销毁。 

- 这⾥就会产⽣问题：当外部类的⽅法结束时，局部变量就会被销毁了，但是内部类对象可能还存在。

- 这⾥就出现了⼀个⽭盾：内部类对象访问了⼀个不存在的变量。

> 为了解决这个问题，就将局部变量复制了⼀份作为内部类的成员变量，这样当局部变量死亡后，内部类仍可以访问它，实际访问的是局部变量的"复制体"。

- 将局部变量设置为final，对它初始化后，我就不让你再去修改这个变量，就保证了内部类的成员变量和⽅法的局部变量的⼀致性。

```java
class Test {
    //局部final变量a,b
    public void test(final int b) {//jdk8在这⾥做了优化, 不⽤写final
        final int a = 10; 
        new Thread() {//匿名内部类,使用了a，b
            public void run() {
                //如果内部类使用了局部变量，这个局部变量必须是final
                System.out.println(a);
                System.out.println(b);
            }
        }.start();
    }
}
```



