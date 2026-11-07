# 范型
## 1. 类型参数命名约定
| 字母 | 含义 |
|------|------|
| `T` | Type（类型） |
| `E` | Element（集合元素） |
| `K` | Key（键） |
| `V` | Value（值） |
| `N` | Number（数值） |
| `R` | Return type（返回类型） |
| `?` | 通配符（Wildcard） |

## 2. 通配符
### 2.1 ? extends T— 只能取，不能放
```java
List<? extends Number> list = new ArrayList<Integer>();
Number n = list.get(0);    // ✅ 取出来一定是 Number
list.add(1);               // ❌ 编译报错
list.add(null);            // ✅ 只有 null 能放
```
为什么不能放？​ 编译器不知道 list实际指向的是 ArrayList<Integer>还是 ArrayList<Double>，放 Integer进去如果实际是 Double列表就炸了 → 编译器直接禁止。

### 2.2 ? super T— 能放，取出来只能是 Object
```java
List<? super Integer> list = new ArrayList<Number>();
list.add(1);               // ✅ Integer 及其子类都能放
list.add(new Integer(2));  // ✅
Object o = list.get(0);    // ✅ 取出来只能是 Object
Integer i = list.get(0);   // ❌ 编译报错
```
为什么取出来是 Object？​ 编译器只知道下限是 Integer，但不知道具体是哪个父类（Number？Object？），所以最安全的类型是 Object。
2.3 PECS 口诀
Producer Extends, Consumer Super
### 2.3 PECS 口诀

> **Producer Extends, Consumer Super**

| 场景 | 写法 | 例子 |
|------|------|------|
| 往外吐数据（生产者） | `? extends T` | `List<? extends Payment>` → 遍历处理 |
| 往里吞数据（消费者） | `? super T` | `List<? super Integer>` → 批量添加 |

---

## 3、类型擦除（面试必考）

### 3.1 擦除规则

| 源码 | 编译后 |
|------|--------|
| `List<String>` | `List` |
| `Box<T>` | `Box<Object>` |
| `Box<T extends Number>` | `Box<Number>` |
| `Map<K, V>` | `Map<Object, Object>` |

### 3.2 由此产生的限制

```java
// ❌ 不能 new 泛型数组
T[] arr = new T[10];

// ❌ 不能 instanceof
if (obj instanceof List<String>)  // 编译报错

// ❌ 不能创建泛型实例
T t = new T();

// ❌ 静态字段不能用类型参数
class Box<T> {
    private static T value;  // ❌
}

// ❌ 重载冲突
void process(List<String> list) {}
void process(List<Integer> list) {}  // ❌ 擦除后签名一样

// ✅ 但泛型方法可以重载（类型参数不同）
<T> void process(List<T> list) {}
<T, U> void process(List<T> list1, List<U> list2) {}  // ✅
```
### 3.3 桥接方法（Bridge Method）
```java
class Node<T> {
    public void setData(T data) { ... }
}

class MyNode extends Node<Integer> {
    @Override
    public void setData(Integer data) { ... }
}
```
擦除后 Node变成 setData(Object)，但 MyNode重写的是 setData(Integer)→ 编译器自动生成一个桥接方法：

```java
// 编译器生成
public void setData(Object data) {
    setData((Integer) data);  // 桥接
}
```

## 3、泛型方法 vs 泛型类
```java
// 泛型类：类型参数在类级别
class Box<T> {
    private T value;
    public T get() { return value; }
}

// 泛型方法：类型参数在方法级别（独立）
class Util {
    public static <T> T max(T a, T b, Comparator<T> c) {
        return c.compare(a, b) > 0 ? a : b;
    }
}
```
