# Java HashMap 主要方法详解

HashMap 是 Java 中最常用的集合类之一，实现了 Map 接口，基于哈希表存储键值对。

## 1. **基本构造方法**

```java
// 1. 默认构造方法 - 初始容量16，负载因子0.75
HashMap<String, Integer> map1 = new HashMap<>();

// 2. 指定初始容量
HashMap<String, Integer> map2 = new HashMap<>(32);

// 3. 指定初始容量和负载因子
HashMap<String, Integer> map3 = new HashMap<>(32, 0.8f);

// 4. 从其他Map构造
Map<String, Integer> existingMap = new HashMap<>();
HashMap<String, Integer> map4 = new HashMap<>(existingMap);
```

## 2. **核心操作方法**

### 2.1 添加/修改元素

```java
HashMap<String, Integer> scores = new HashMap<>();

// 1. put() - 添加或更新键值对
scores.put("Alice", 95);      // 添加 Alice -> 95
scores.put("Bob", 88);        // 添加 Bob -> 88
scores.put("Alice", 98);      // 更新 Alice -> 98，返回旧值95

// 2. putIfAbsent() - 仅当键不存在时插入
scores.putIfAbsent("Bob", 90);     // Bob已存在，不更新
scores.putIfAbsent("Charlie", 85); // Charlie不存在，插入85

// 3. putAll() - 批量添加
Map<String, Integer> newScores = new HashMap<>();
newScores.put("David", 92);
newScores.put("Eve", 89);
scores.putAll(newScores);
```

### 2.2 获取元素

```java
// 1. get() - 根据键获取值
Integer aliceScore = scores.get("Alice");     // 98
Integer unknown = scores.get("Unknown");      // null

// 2. getOrDefault() - 获取值，不存在时返回默认值
Integer score = scores.getOrDefault("Frank", 60); // Frank不存在，返回60

// 3. containsKey() - 检查是否包含键
boolean hasAlice = scores.containsKey("Alice");   // true
boolean hasFrank = scores.containsKey("Frank");   // false

// 4. containsValue() - 检查是否包含值
boolean hasScore98 = scores.containsValue(98);    // true
```

### 2.3 删除元素

```java
// 1. remove(key) - 根据键删除
Integer removed = scores.remove("Bob");  // 删除Bob，返回88

// 2. remove(key, value) - 仅当键值都匹配时删除
boolean isRemoved = scores.remove("Alice", 95);   // false，因为Alice的值是98
boolean isRemoved2 = scores.remove("Alice", 98);  // true

// 3. clear() - 清空所有元素
scores.clear();  // 清空HashMap
```

## 3. **遍历方法**

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("Apple", 10);
map.put("Banana", 20);
map.put("Cherry", 30);

// 1. 遍历键集合
System.out.println("Keys:");
for (String key : map.keySet()) {
    System.out.println(key);
}

// 2. 遍历值集合
System.out.println("Values:");
for (Integer value : map.values()) {
    System.out.println(value);
}

// 3. 遍历键值对（Entry）
System.out.println("Entries:");
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    Integer value = entry.getValue();
    System.out.println(key + " = " + value);
}

// 4. 使用迭代器
System.out.println("Using Iterator:");
Iterator<Map.Entry<String, Integer>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry<String, Integer> entry = iterator.next();
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// 5. Java 8+ forEach方法
System.out.println("Using forEach:");
map.forEach((key, value) -> System.out.println(key + " -> " + value));
```

## 4. **Java 8+ 新增方法**

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

// 1. compute() - 计算新值
map.compute("A", (key, value) -> value * 10);  // A: 1 → 10

// 2. computeIfAbsent() - 键不存在时计算新值
map.computeIfAbsent("D", key -> 4);  // D不存在，添加 D=4
map.computeIfAbsent("A", key -> 100); // A已存在，不改变

// 3. computeIfPresent() - 键存在时计算新值
map.computeIfPresent("B", (key, value) -> value + 10);  // B: 2 → 12
map.computeIfPresent("E", (key, value) -> 5);  // E不存在，不操作

// 4. merge() - 合并值
map.merge("C", 10, (oldValue, newValue) -> oldValue + newValue);  // C: 3 → 13
map.merge("F", 5, (oldValue, newValue) -> oldValue + newValue);   // F不存在，添加 F=5

// 5. replaceAll() - 替换所有值
map.replaceAll((key, value) -> value * 2);  // 所有值乘以2

// 6. getOrDefault() 的增强用法
Integer value = map.getOrDefault("Z", 0);  // Z不存在，返回0
```

## 5. **其他实用方法**

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("One", 1);
map.put("Two", 2);
map.put("Three", 3);

// 1. size() - 获取元素数量
int size = map.size();  // 3

// 2. isEmpty() - 判断是否为空
boolean empty = map.isEmpty();  // false

// 3. clone() - 浅拷贝
HashMap<String, Integer> cloned = (HashMap<String, Integer>) map.clone();

// 4. replace() - 替换值
Integer oldValue = map.replace("Two", 22);  // 返回旧值2
boolean replaced = map.replace("Three", 3, 33);  // 只有旧值为3时才替换

// 5. toString() - 转换为字符串
String mapStr = map.toString();  // {One=1, Two=22, Three=33}
```

## 6. **线程安全处理**

```java
// HashMap不是线程安全的
HashMap<String, Integer> unsafeMap = new HashMap<>();

// 1. 使用Collections.synchronizedMap包装
Map<String, Integer> synchronizedMap = 
    Collections.synchronizedMap(new HashMap<>());

// 使用示例
synchronized(synchronizedMap) {
    synchronizedMap.put("key", "value");
}

// 2. 使用ConcurrentHashMap（推荐）
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("key", 1);
```

## 7. **完整示例**

```java
import java.util.*;

public class HashMapExample {
    public static void main(String[] args) {
        // 创建HashMap
        HashMap<String, Student> studentMap = new HashMap<>();
        
        // 添加元素
        studentMap.put("S001", new Student("Alice", 20));
        studentMap.put("S002", new Student("Bob", 21));
        studentMap.put("S003", new Student("Charlie", 22));
        
        // 获取元素
        Student alice = studentMap.get("S001");
        System.out.println("S001: " + alice);
        
        // 检查键是否存在
        if (studentMap.containsKey("S002")) {
            System.out.println("S002 exists");
        }
        
        // 遍历HashMap
        System.out.println("\nAll students:");
        for (Map.Entry<String, Student> entry : studentMap.entrySet()) {
            System.out.println("ID: " + entry.getKey() + 
                             ", Student: " + entry.getValue());
        }
        
        // 使用Java 8+方法
        studentMap.computeIfPresent("S001", (id, student) -> {
            student.setAge(student.getAge() + 1);
            return student;
        });
        
        // 统计信息
        System.out.println("\nTotal students: " + studentMap.size());
        
        // 删除元素
        Student removed = studentMap.remove("S003");
        System.out.println("Removed: " + removed);
        
        // 清空
        studentMap.clear();
        System.out.println("Is empty: " + studentMap.isEmpty());
    }
}

class Student {
    private String name;
    private int age;
    
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    
    @Override
    public String toString() {
        return name + " (" + age + ")";
    }
}
```

## 8. **性能注意事项**

```java
public class HashMapPerformance {
    public static void main(String[] args) {
        // 1. 初始容量设置
        // 如果知道大致数量，设置初始容量避免扩容
        int expectedSize = 1000;
        HashMap<String, String> map1 = new HashMap<>(expectedSize);
        
        // 2. 负载因子权衡
        // 0.75是时间空间的平衡点
        HashMap<String, String> map2 = new HashMap<>(16, 0.75f);
        
        // 3. 遍历性能比较
        HashMap<Integer, String> largeMap = new HashMap<>();
        for (int i = 0; i < 100000; i++) {
            largeMap.put(i, "Value" + i);
        }
        
        // 遍历方式影响性能
        long startTime = System.currentTimeMillis();
        // 方式1：遍历EntrySet（推荐）
        for (Map.Entry<Integer, String> entry : largeMap.entrySet()) {
            Integer key = entry.getKey();
            String value = entry.getValue();
        }
        
        // 方式2：遍历KeySet（不推荐，需要额外get）
        for (Integer key : largeMap.keySet()) {
            String value = largeMap.get(key);
        }
        
        // 4. 重写hashCode和equals
        // 作为HashMap的键的对象必须正确重写hashCode和equals方法
    }
}
```

## 9. **常见问题与技巧**

```java
public class HashMapTips {
    public static void main(String[] args) {
        // 1. null键和null值
        HashMap<String, String> map = new HashMap<>();
        map.put(null, "Null Key");      // 允许null键
        map.put("Key", null);           // 允许null值
        
        // 2. 值类型转换
        HashMap<String, Object> objMap = new HashMap<>();
        objMap.put("count", 10);
        objMap.put("name", "John");
        
        // 安全获取并转换
        Object countObj = objMap.get("count");
        if (countObj instanceof Integer) {
            int count = (Integer) countObj;
        }
        
        // 3. 使用entrySet()遍历时删除元素
        HashMap<String, Integer> scores = new HashMap<>();
        scores.put("A", 50);
        scores.put("B", 60);
        scores.put("C", 70);
        
        Iterator<Map.Entry<String, Integer>> it = scores.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<String, Integer> entry = it.next();
            if (entry.getValue() < 60) {
                it.remove();  // 安全删除
            }
        }
        
        // 4. 使用computeIfAbsent初始化复杂值
        HashMap<String, List<String>> multiMap = new HashMap<>();
        multiMap.computeIfAbsent("group1", k -> new ArrayList<>())
                .add("item1");
    }
}
```

## 主要特点总结：

1. **允许null键和null值**
2. **非线程安全**，需要外部同步
3. **不保证元素顺序**（LinkedHashMap保持插入顺序，TreeMap有序）
4. **初始容量16**，负载因子0.75
5. **键对象必须正确实现hashCode()和equals()**
6. **Java 8+引入了红黑树优化**，链表长度超过8时转换为红黑树

HashMap是最常用的Map实现，适用于大多数需要键值对存储的场景。