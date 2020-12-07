---
title: Effective Java Note
---

# 1.考虑使用静态工厂方法替代构造器
* 静态工厂VS构造器优点:
  1. 它们有名称  
  2. 不必再每次调用时都创建对象  
  3. 可以返回原类型的任何子类型的对象
  4. 所返回的对象的类可以随着每次调用而发生变化, 这取决于静态工厂方法的参数值
  5. 方法所返回的对象所属的类, 在编写该静态工厂方法时可以不存在
* 静态工厂VS构造器缺点:
  1. 类如果没有public或者default的构造器, 子类将不会被实例化  
  2. 它们与其它静态方法没有区别, 不会像构造器那样被标注出来  

# 2.遇到多个构造器参数时要考虑使用构建器(建造者模式)
JavaBeans的多个set模式, 无法保证对象处于一致的状态, 也就无法保证线程安全

# 3.用私有构造器或者枚举类型强化Singleton属性 
* 实现方式
  1. 公有静态成员  
  2. 静态工厂  
  3. 单元素枚举类  

# 4.通过私有构造器来强化不可实例化的能力
副作用:不能子类化, 子类没有可访问的超类构造方法

# 5.优先考虑依赖注入来引用资源
静态工厂和建造者模式不适合依赖于底层资源的类的实力化
```
public class SpellChecke ｛
  private final Lexicon dictiona y;
  // 依赖注入
  public SpellChecker(Lexicon dictionary) {  
    this.dictiona = Objects eq eNonNull(dictio y);
  }
  public boolean isValid(St ng WO { . . . } 
  public List<String> suggestions(String typo) { .. . }
}
```
# 6.避免创建不必要的对象
* 这将创建2个功能方面完全等同的对象
    ```
    String s = new String("Hello World");
    ```
* 应该是常量的对象被反复创建
* 自动拆装箱的过程中(求和操作) 使用了包装类型 创建了不必要的对象

# 7.消除过期对象引用
* 自己维护一个数据结构，里面持有实际的内存（比如array） 需要把删除的对象的引用置为空，来GC
* 缓存的时候 使用weakhashmap 当key的引用为空的时候 该key value会被删除
* 注册监听器的时候使用弱引用

# 8.避免使用终结方法
1. 不可预测
2. 非常严重的性能损失
3. 使用场景
  * 作为最后一道安全网
  * 终止非关键的本地资源
  * 如果使用了finalize方法记得调用super.finalize方法；或者使用终结方法守卫者（私有的Object对象，只覆写了finalize方法）来保证调用到

# 9.try-with-resources优先于try-finally
try(资源)catch(Exception)
使代码更简洁, 清晰 ,资源自动释放(前提是资源实现了AutoCloseable接口)

# 10.覆盖equals方法时遵守约定
* 以下不需要覆写
  1. 类的每个实例本质本质就是唯一的
  2. 并不关心类是否提供了逻辑相等
  3. 超类已经覆写了equals，从超类继承的行为对于子类也是合适的
  4. 类是私有的，或者包级私有的， 这种情况下equals方法永远不会被调用
  5. 单例模式不需要覆写
* 覆写需要满足的约定
  1. 自反性 x equals x = true
  2. 对称性 x equals y = true y equals x = true
  3. 传递性 x equals y = true y equals z = true x equals z = true
  4. 一致性 x equals y = true equals方法里面的信息没有被修改 多次调用始终成立
  5. null x euqals null = false
* 在子类继承破坏了传递性的时候的解决措施
  1. getClass来判断运行时类信息, 不一致直接false
  2. 使用组合来代替继承, 适时返回view对象(要比较的对象)
  3. 父类没必要实例化的时候可以是abstract类
* equals方法高质量诀窍：
  1. 性能优化 使用==来检查参数是否是this本身
  2. 使用instanceof操作符来检查参数是否是正确的类型
  3. 把参数转换成正确的类型
  4. 对该类中每个关键的域，检查参数中的域与该对象的域是否匹配
      1. double使用Double.compare
      2. float使用Float.compare
      3. 对于数组域使用Arrays.equals方法
      4. 避免NPE （field == null ? o.field == null : field.equals(o.field))  或者 （field == o.field ||(field != null && field.equals(o.field)))
      5. 优先比较最有可能不一致的域或者开销最低的域
  5. 编写结束后进行单元测试
      1. 覆盖equals方法时总要覆盖hashcode方法
      2. 不要企图让equals方法过于智能
      3. 不要更改Object类型为别的 （此操作为重载， 非覆写）
      
# 11.覆盖equals方法时总要覆盖hashcode方法
如果类的成员变量全是final的，考虑使用volatile和懒加载将hashcode的值缓存起来

# 12.始终要覆盖toString方法

# 13.谨慎地覆盖clone（不要使用）X
最好使用clone来复制Array

# 14.考虑实现Comparable接口
可以考虑定义一个类的比较器, 调用比较器的compare方法来比较

# 15.使类和成员的可访问性最小化
尽可能地使每个类或者成员不被外界访问  
1. 如果一个类只被一个类访问,考虑把该类变成访问类的内部私有嵌套类
2. 共有类的实例域绝不能是共有的,包含共有可变域的类通常是线程不安全的
3. 注意定长的数组，即使是final的也是可变的
  
# 16.要在共有类而非共有域中使用访问方法
1. 如果类可以在包外被访问,就提供访问方法
2. 如果类是包级私有或者私有的嵌套类,直接暴露它的数据域并没有本质的错误

# 17.使可变性最小化
* 遵循的原则:
  1. 不要提供任何设值方法
  2. 保证类不会被拓展
  3. 声明所有的域都是final的
  4. 声明所有的域都是私有的
  5. 确保对于任何可变组件的互斥访问
* 优点:
  1. 不可变对象简单
  2. 不可变对象本质是线程安全的
  3. 不可变对象可以被自由地贡献
  4. 不仅可以共享不可变对象,也可以共享它们的内部信息
  5. 不可变对象为其他对象提供了大量的控件
  6. 不可变对象无偿的提供了失败的原子性
* 缺点:
  1. 对于每一个不同的值都需要一个不同的对象
* 注意点:
  1. 如果不可变的类实现了序列化接口, 且包含一个或多个指向可变对象的域, 就必须提供一个显式的readObject或者readResolve方法, 或者使用ObjectOutputStream.writeUnshared和ObjectInputStream.readUnshared方法
  2. 不要为每个get方法写set方法, 除非有很好的理由让类变成可变的类
  3. 如果类不是不可变的, 仍应该限制其可变性---即private final
  4. 构造器应该创建完全初始化的对象, 并建立起所有的约束关系
  
# 18.复合优于继承
  
  
  
  
  
  
  
