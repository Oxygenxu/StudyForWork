# Java Arrays 工具类详解

`java.util.Arrays` 是 Java 中处理数组的核心工具类，提供了丰富的静态方法来操作数组。它位于 `java.util` 包中，自 JDK 1.2 引入。

## 1. **基本介绍**

### 1.1 类声明
```java
public class Arrays {
    // 私有构造方法，防止实例化
    private Arrays() {}
    
    // 全部为静态方法
    public static void sort(int[] a) { ... }
    public static int binarySearch(int[] a, int key) { ... }
    // ... 其他方法
}
```

### 1.2 主要特点
- 所有方法都是静态的，通过类名直接调用
- 支持所有基本类型数组和对象数组
- 方法命名规范，功能明确

## 2. **排序相关方法**

### 2.1 基本排序
```java
import java.util.Arrays;

public class SortExample {
    public static void main(String[] args) {
        // 1. sort() - 对整个数组排序（升序）
        int[] numbers = {5, 2, 8, 1, 9};
        Arrays.sort(numbers);
        System.out.println("Sorted: " + Arrays.toString(numbers));
        // 输出: [1, 2, 5, 8, 9]
        
        // 2. 部分排序
        int[] arr = {5, 2, 8, 1, 9, 3, 7};
        Arrays.sort(arr, 2, 5);  // 对索引[2,5)的元素排序
        System.out.println("Partially sorted: " + Arrays.toString(arr));
        // 输出: [5, 2, 1, 8, 9, 3, 7]
        
        // 3. 对象数组排序
        String[] names = {"Bob", "Alice", "David", "Charlie"};
        Arrays.sort(names);
        System.out.println("Sorted strings: " + Arrays.toString(names));
        // 输出: [Alice, Bob, Charlie, David]
    }
}
```

### 2.2 自定义排序
```java
import java.util.Arrays;
import java.util.Comparator;

class Person {
    String name;
    int age;
    
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return name + "(" + age + ")";
    }
}

public class CustomSortExample {
    public static void main(String[] args) {
        Person[] people = {
            new Person("Alice", 25),
            new Person("Bob", 20),
            new Person("Charlie", 30),
            new Person("David", 22)
        };
        
        // 1. 按年龄排序
        Arrays.sort(people, new Comparator<Person>() {
            @Override
            public int compare(Person p1, Person p2) {
                return Integer.compare(p1.age, p2.age);
            }
        });
        System.out.println("By age: " + Arrays.toString(people));
        
        // 2. 使用Lambda表达式（Java 8+）
        Arrays.sort(people, (p1, p2) -> p1.name.compareTo(p2.name));
        System.out.println("By name: " + Arrays.toString(people));
        
        // 3. 使用Comparator.comparing
        Arrays.sort(people, Comparator.comparing(p -> p.age));
        Arrays.sort(people, Comparator.comparing(p -> p.name));
    }
}
```

### 2.3 并行排序（Java 8+）
```java
import java.util.Arrays;
import java.util.Random;

public class ParallelSortExample {
    public static void main(String[] args) {
        // 创建大型数组
        int size = 1000000;
        int[] largeArray = new int[size];
        Random random = new Random();
        
        for (int i = 0; i < size; i++) {
            largeArray[i] = random.nextInt(1000000);
        }
        
        // 复制数组用于比较
        int[] copy1 = Arrays.copyOf(largeArray, size);
        int[] copy2 = Arrays.copyOf(largeArray, size);
        
        // 1. 普通排序
        long start = System.currentTimeMillis();
        Arrays.sort(copy1);
        long end = System.currentTimeMillis();
        System.out.println("Arrays.sort() time: " + (end - start) + "ms");
        
        // 2. 并行排序
        start = System.currentTimeMillis();
        Arrays.parallelSort(copy2);
        end = System.currentTimeMillis();
        System.out.println("Arrays.parallelSort() time: " + (end - start) + "ms");
        
        // 验证结果相同
        System.out.println("Arrays equal: " + Arrays.equals(copy1, copy2));
    }
}
```

## 3. **查找相关方法**

### 3.1 二分查找
```java
import java.util.Arrays;

public class BinarySearchExample {
    public static void main(String[] args) {
        // 数组必须已排序
        int[] numbers = {10, 20, 30, 40, 50, 60, 70, 80, 90};
        
        // 1. 查找存在的元素
        int index = Arrays.binarySearch(numbers, 40);
        System.out.println("Index of 40: " + index);  // 3
        
        // 2. 查找不存在的元素
        index = Arrays.binarySearch(numbers, 35);
        System.out.println("Index of 35: " + index);  // 负数，表示插入点
        
        // 3. 部分范围查找
        index = Arrays.binarySearch(numbers, 2, 7, 60);  // 在索引[2,7)范围内查找
        System.out.println("Index of 60 in range: " + index);  // 5
        
        // 4. 对象数组查找
        String[] names = {"Alice", "Bob", "Charlie", "David"};
        Arrays.sort(names);  // 必须先排序
        index = Arrays.binarySearch(names, "Charlie");
        System.out.println("Index of Charlie: " + index);  // 2
    }
}
```

### 3.2 二分查找返回值规则
```java
int[] arr = {10, 20, 30, 40, 50};

// 规则：
// 1. 如果找到，返回元素索引（从0开始）
System.out.println(Arrays.binarySearch(arr, 30));  // 2

// 2. 如果没找到，返回 -(插入点) - 1
// 插入点：第一个大于key的元素的索引，或数组长度（如果所有元素都小于key）
System.out.println(Arrays.binarySearch(arr, 25));  // -3
// 解释：插入点为2，返回 -2-1 = -3

System.out.println(Arrays.binarySearch(arr, 5));   // -1 (插入点0)
System.out.println(Arrays.binarySearch(arr, 60));  // -6 (插入点5，即arr.length)
```

## 4. **比较相关方法**

### 4.1 equals() 方法
```java
import java.util.Arrays;

public class EqualsExample {
    public static void main(String[] args) {
        // 1. 基本类型数组比较
        int[] arr1 = {1, 2, 3};
        int[] arr2 = {1, 2, 3};
        int[] arr3 = {1, 2, 4};
        
        System.out.println("arr1 equals arr2: " + Arrays.equals(arr1, arr2));  // true
        System.out.println("arr1 equals arr3: " + Arrays.equals(arr1, arr3));  // false
        
        // 2. 对象数组比较（比较引用）
        String[] str1 = {"A", "B"};
        String[] str2 = {"A", "B"};
        String[] str3 = {"A", "C"};
        
        System.out.println("str1 equals str2: " + Arrays.equals(str1, str2));  // true
        System.out.println("str1 == str2: " + (str1 == str2));                 // false
        
        // 3. 比较部分范围
        int[] arr4 = {1, 2, 3, 4, 5};
        int[] arr5 = {9, 2, 3, 8, 10};
        boolean result = Arrays.equals(arr4, 1, 3, arr5, 1, 3);  // 比较arr4[1,3)和arr5[1,3)
        System.out.println("Partial equals: " + result);  // true
    }
}
```

### 4.2 deepEquals() 方法
```java
import java.util.Arrays;

public class DeepEqualsExample {
    public static void main(String[] args) {
        // 用于嵌套数组或对象数组的深层比较
        int[][] matrix1 = {{1, 2}, {3, 4}};
        int[][] matrix2 = {{1, 2}, {3, 4}};
        int[][] matrix3 = {{1, 2}, {3, 5}};
        
        System.out.println("Arrays.equals: " + Arrays.equals(matrix1, matrix2));  // false
        System.out.println("Arrays.deepEquals: " + Arrays.deepEquals(matrix1, matrix2));  // true
        System.out.println("Arrays.deepEquals: " + Arrays.deepEquals(matrix1, matrix3));  // false
        
        // 对象数组
        String[][] names1 = {{"Alice", "Bob"}, {"Charlie", "David"}};
        String[][] names2 = {{"Alice", "Bob"}, {"Charlie", "David"}};
        String[][] names3 = {{"Alice", "Bob"}, {"Charlie", "Eve"}};
        
        System.out.println("deepEquals names1 & names2: " + 
                          Arrays.deepEquals(names1, names2));  // true
        System.out.println("deepEquals names1 & names3: " + 
                          Arrays.deepEquals(names1, names3));  // false
    }
}
```

### 4.3 compare() 方法（Java 9+）
```java
import java.util.Arrays;

public class CompareExample {
    public static void main(String[] args) {
        // 1. 比较基本类型数组
        int[] arr1 = {1, 2, 3};
        int[] arr2 = {1, 2, 3};
        int[] arr3 = {1, 2, 4};
        int[] arr4 = {1, 2, 2};
        
        System.out.println("compare arr1, arr2: " + Arrays.compare(arr1, arr2));  // 0 (相等)
        System.out.println("compare arr1, arr3: " + Arrays.compare(arr1, arr3));  // -1 (arr1 < arr3)
        System.out.println("compare arr3, arr1: " + Arrays.compare(arr3, arr1));  // 1 (arr3 > arr1)
        System.out.println("compare arr1, arr4: " + Arrays.compare(arr1, arr4));  // 1 (arr1 > arr4)
        
        // 2. 比较部分范围
        int[] arr5 = {1, 2, 3, 4, 5};
        int[] arr6 = {0, 2, 3, 9, 0};
        int result = Arrays.compare(arr5, 1, 3, arr6, 1, 3);
        System.out.println("Partial compare: " + result);  // 0 (相等)
        
        // 3. 比较规则
        // 返回负整数: 第一个数组小于第二个
        // 返回0: 相等
        // 返回正整数: 第一个数组大于第二个
    }
}
```

## 5. **复制相关方法**

### 5.1 copyOf() 方法
```java
import java.util.Arrays;

public class CopyOfExample {
    public static void main(String[] args) {
        // 1. 基本复制
        int[] original = {1, 2, 3, 4, 5};
        int[] copy = Arrays.copyOf(original, original.length);
        System.out.println("Copy: " + Arrays.toString(copy));
        
        // 2. 截断复制
        int[] truncated = Arrays.copyOf(original, 3);
        System.out.println("Truncated: " + Arrays.toString(truncated));  // [1, 2, 3]
        
        // 3. 扩展复制
        int[] extended = Arrays.copyOf(original, 8);
        System.out.println("Extended: " + Arrays.toString(extended));  // [1, 2, 3, 4, 5, 0, 0, 0]
        
        // 4. 对象数组复制（浅拷贝）
        String[] names = {"Alice", "Bob", "Charlie"};
        String[] namesCopy = Arrays.copyOf(names, names.length);
        namesCopy[0] = "David";  // 不影响原数组
        System.out.println("Original: " + Arrays.toString(names));      // [Alice, Bob, Charlie]
        System.out.println("Copy: " + Arrays.toString(namesCopy));      // [David, Bob, Charlie]
    }
}
```

### 5.2 copyOfRange() 方法
```java
import java.util.Arrays;

public class CopyOfRangeExample {
    public static void main(String[] args) {
        int[] original = {10, 20, 30, 40, 50, 60, 70};
        
        // 1. 复制指定范围
        int[] range1 = Arrays.copyOfRange(original, 2, 5);  // 索引[2,5)
        System.out.println("Range [2,5): " + Arrays.toString(range1));  // [30, 40, 50]
        
        // 2. 超出原数组范围
        int[] range2 = Arrays.copyOfRange(original, 5, 10);  // 超出部分用0填充
        System.out.println("Range [5,10): " + Arrays.toString(range2));  // [60, 70, 0, 0, 0]
        
        // 3. 对象数组
        String[] fruits = {"Apple", "Banana", "Cherry", "Date", "Elderberry"};
        String[] sliced = Arrays.copyOfRange(fruits, 1, 4);
        System.out.println("Fruits [1,4): " + Arrays.toString(sliced));  // [Banana, Cherry, Date]
    }
}
```

### 5.3 System.arraycopy() 对比
```java
import java.util.Arrays;

public class ArrayCopyComparison {
    public static void main(String[] args) {
        int[] source = {1, 2, 3, 4, 5};
        int[] dest1 = new int[5];
        int[] dest2 = new int[5];
        
        // 1. 使用System.arraycopy（底层方法，性能更好）
        System.arraycopy(source, 0, dest1, 0, source.length);
        System.out.println("System.arraycopy: " + Arrays.toString(dest1));
        
        // 2. 使用Arrays.copyOf（更简洁）
        dest2 = Arrays.copyOf(source, source.length);
        System.out.println("Arrays.copyOf: " + Arrays.toString(dest2));
        
        // 比较：
        // System.arraycopy: 需要先创建目标数组
        // Arrays.copyOf: 自动创建目标数组，更简洁
    }
}
```

## 6. **填充和设置方法**

### 6.1 fill() 方法
```java
import java.util.Arrays;

public class FillExample {
    public static void main(String[] args) {
        // 1. 全部填充
        int[] arr1 = new int[5];
        Arrays.fill(arr1, 7);
        System.out.println("Filled with 7: " + Arrays.toString(arr1));  // [7, 7, 7, 7, 7]
        
        // 2. 部分填充
        int[] arr2 = new int[10];
        Arrays.fill(arr2, 3, 7, 99);  // 填充索引[3,7)的元素
        System.out.println("Partially filled: " + Arrays.toString(arr2));
        // [0, 0, 0, 99, 99, 99, 99, 0, 0, 0]
        
        // 3. 对象数组填充
        String[] strings = new String[4];
        Arrays.fill(strings, "Hello");
        System.out.println("String array: " + Arrays.toString(strings));
        // [Hello, Hello, Hello, Hello]
        
        // 4. 填充引用类型（所有元素引用同一个对象）
        Person[] people = new Person[3];
        Arrays.fill(people, new Person("Default", 0));
        // 注意：所有数组元素引用的是同一个Person对象
    }
}
```

### 6.2 setAll() 和 parallelSetAll()（Java 8+）
```java
import java.util.Arrays;
import java.util.Random;

public class SetAllExample {
    public static void main(String[] args) {
        // 1. setAll() - 使用函数设置所有元素
        int[] arr1 = new int[5];
        Arrays.setAll(arr1, i -> i * 2);  // i是索引
        System.out.println("setAll (i*2): " + Arrays.toString(arr1));  // [0, 2, 4, 6, 8]
        
        // 2. 复杂计算
        double[] doubles = new double[5];
        Arrays.setAll(doubles, i -> Math.sin(i * 0.5));
        System.out.println("setAll (sin): " + Arrays.toString(doubles));
        
        // 3. 随机数
        Random random = new Random();
        int[] randomArr = new int[10];
        Arrays.setAll(randomArr, i -> random.nextInt(100));
        System.out.println("setAll (random): " + Arrays.toString(randomArr));
        
        // 4. parallelSetAll() - 并行设置（适用于大数组）
        int[] largeArray = new int[1000000];
        Arrays.parallelSetAll(largeArray, i -> i * i);
        
        // 比较性能
        int size = 10000000;
        int[] arr2 = new int[size];
        int[] arr3 = new int[size];
        
        long start = System.currentTimeMillis();
        Arrays.setAll(arr2, i -> i * i);
        long end = System.currentTimeMillis();
        System.out.println("setAll time: " + (end - start) + "ms");
        
        start = System.currentTimeMillis();
        Arrays.parallelSetAll(arr3, i -> i * i);
        end = System.currentTimeMillis();
        System.out.println("parallelSetAll time: " + (end - start) + "ms");
    }
}
```

## 7. **转换相关方法**

### 7.1 toString() 和 deepToString()
```java
import java.util.Arrays;

public class ToStringExample {
    public static void main(String[] args) {
        // 1. toString() - 基本类型和对象数组
        int[] numbers = {1, 2, 3, 4, 5};
        System.out.println("toString: " + Arrays.toString(numbers));
        // 输出: [1, 2, 3, 4, 5]
        
        String[] names = {"Alice", "Bob", "Charlie"};
        System.out.println("String array: " + Arrays.toString(names));
        // 输出: [Alice, Bob, Charlie]
        
        // 2. deepToString() - 嵌套数组
        int[][] matrix = {{1, 2}, {3, 4}, {5, 6}};
        System.out.println("Arrays.toString(matrix): " + Arrays.toString(matrix));
        // 输出: [[I@哈希码, [I@哈希码, [I@哈希码]
        
        System.out.println("Arrays.deepToString(matrix): " + Arrays.deepToString(matrix));
        // 输出: [[1, 2], [3, 4], [5, 6]]
        
        // 3. 多层嵌套
        int[][][] cube = {
            {{1, 2}, {3, 4}},
            {{5, 6}, {7, 8}}
        };
        System.out.println("deepToString 3D: " + Arrays.deepToString(cube));
        // 输出: [[[1, 2], [3, 4]], [[5, 6], [7, 8]]]
    }
}
```

### 7.2 asList() 方法
```java
import java.util.Arrays;
import java.util.List;

public class AsListExample {
    public static void main(String[] args) {
        // 1. 基本用法
        String[] array = {"Apple", "Banana", "Cherry"};
        List<String> list = Arrays.asList(array);
        System.out.println("List: " + list);
        // 输出: [Apple, Banana, Cherry]
        
        // 2. 注意：返回的List是固定大小的
        // list.add("Date");  // 抛出UnsupportedOperationException
        // list.remove(0);     // 抛出UnsupportedOperationException
        
        // 但可以修改元素
        list.set(0, "Apricot");
        System.out.println("Modified list: " + list);
        System.out.println("Original array: " + Arrays.toString(array));
        // 原数组也被修改：数组和List共享数据
        
        // 3. 直接使用可变参数
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        System.out.println("Numbers: " + numbers);
        
        // 4. 转换为可修改的List
        List<String> modifiableList = new java.util.ArrayList<>(Arrays.asList(array));
        modifiableList.add("Date");  // 现在可以添加
        System.out.println("Modifiable list: " + modifiableList);
    }
}
```

### 7.3 stream() 方法（Java 8+）
```java
import java.util.Arrays;

public class StreamExample {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        
        // 1. 创建流
        int sum = Arrays.stream(numbers).sum();
        System.out.println("Sum: " + sum);
        
        // 2. 流操作
        double average = Arrays.stream(numbers).average().orElse(0);
        System.out.println("Average: " + average);
        
        // 3. 过滤和转换
        Arrays.stream(numbers)
              .filter(n -> n % 2 == 0)        // 过滤偶数
              .map(n -> n * n)                // 平方
              .forEach(System.out::println);  // 打印
        
        // 4. 部分流
        Arrays.stream(numbers, 3, 7)  // 索引[3,7)的元素
              .forEach(System.out::println);
        
        // 5. 对象数组流
        String[] names = {"Alice", "Bob", "Charlie", "David"};
        long count = Arrays.stream(names)
                          .filter(name -> name.length() > 4)
                          .count();
        System.out.println("Names longer than 4 chars: " + count);
    }
}
```

## 8. **其他实用方法**

### 8.1 spliterator() 方法（Java 8+）
```java
import java.util.Arrays;
import java.util.Spliterator;

public class SpliteratorExample {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        
        // 1. 获取Spliterator
        Spliterator.OfInt spliterator = Arrays.spliterator(numbers);
        
        // 2. 遍历
        spliterator.forEachRemaining((int value) -> 
            System.out.print(value + " "));
        System.out.println();
        
        // 3. 分割（用于并行处理）
        Spliterator.OfInt split1 = Arrays.spliterator(numbers, 0, numbers.length / 2);
        Spliterator.OfInt split2 = Arrays.spliterator(numbers, numbers.length / 2, numbers.length);
        
        System.out.println("Split1:");
        split1.forEachRemaining(System.out::print);
        System.out.println("\nSplit2:");
        split2.forEachRemaining(System.out::print);
    }
}
```

### 8.2 hashCode() 和 deepHashCode()
```java
import java.util.Arrays;

public class HashCodeExample {
    public static void main(String[] args) {
        // 1. hashCode() - 数组的哈希码
        int[] arr1 = {1, 2, 3};
        int[] arr2 = {1, 2, 3};
        System.out.println("arr1 hashCode: " + Arrays.hashCode(arr1));
        System.out.println("arr2 hashCode: " + Arrays.hashCode(arr2));
        // 相同内容的数组有相同的哈希码
        
        // 2. deepHashCode() - 深层哈希码
        int[][] matrix1 = {{1, 2}, {3, 4}};
        int[][] matrix2 = {{1, 2}, {3, 4}};
        System.out.println("matrix1 hashCode: " + Arrays.hashCode(matrix1));
        System.out.println("matrix2 hashCode: " + Arrays.hashCode(matrix2));
        System.out.println("matrix1 deepHashCode: " + Arrays.deepHashCode(matrix1));
        System.out.println("matrix2 deepHashCode: " + Arrays.deepHashCode(matrix2));
        // hashCode() 不同（比较引用）
        // deepHashCode() 相同（比较内容）
    }
}
```

## 9. **性能优化技巧**

```java
import java.util.Arrays;

public class PerformanceTips {
    public static void main(String[] args) {
        // 1. 对于小型数组，使用Arrays.sort()
        // 2. 对于大型数组(>1000)，使用Arrays.parallelSort()
        
        // 3. 频繁查找时，先排序再用binarySearch
        int[] data = {5, 3, 8, 1, 9, 2, 7, 4, 6};
        Arrays.sort(data);  // 先排序
        int index = Arrays.binarySearch(data, 7);  // 快速查找
        
        // 4. 使用Arrays.copyOf而不是循环复制
        int[] source = {1, 2, 3, 4, 5};
        // 不好：
        int[] dest1 = new int[source.length];
        for (int i = 0; i < source.length; i++) {
            dest1[i] = source[i];
        }
        // 好：
        int[] dest2 = Arrays.copyOf(source, source.length);
        
        // 5. 使用Arrays.fill初始化数组
        // 不好：
        int[] arr1 = new int[100];
        for (int i = 0; i < arr1.length; i++) {
            arr1[i] = -1;
        }
        // 好：
        int[] arr2 = new int[100];
        Arrays.fill(arr2, -1);
    }
}
```

## 10. **完整示例**

```java
import java.util.Arrays;
import java.util.Comparator;

public class ArraysCompleteExample {
    public static void main(String[] args) {
        // 创建测试数据
        int[] numbers = {5, 2, 8, 1, 9, 3, 7, 4, 6};
        String[] names = {"Alice", "Bob", "Charlie", "David", "Eve"};
        
        System.out.println("原始数组:");
        System.out.println("numbers: " + Arrays.toString(numbers));
        System.out.println("names: " + Arrays.toString(names));
        
        // 排序
        Arrays.sort(numbers);
        Arrays.sort(names, Comparator.reverseOrder());
        
        System.out.println("\n排序后:");
        System.out.println("numbers: " + Arrays.toString(numbers));
        System.out.println("names: " + Arrays.toString(names));
        
        // 查找
        int index = Arrays.binarySearch(numbers, 7);
        System.out.println("\n数字7的索引: " + index);
        
        // 复制
        int[] copy = Arrays.copyOf(numbers, 5);
        System.out.println("前5个数字: " + Arrays.toString(copy));
        
        // 填充
        int[] filled = new int[5];
        Arrays.fill(filled, 42);
        System.out.println("填充数组: " + Arrays.toString(filled));
        
        // 比较
        int[] arr1 = {1, 2, 3};
        int[] arr2 = {1, 2, 3};
        System.out.println("\n数组比较:");
        System.out.println("Arrays.equals: " + Arrays.equals(arr1, arr2));
        System.out.println("Arrays.compare: " + Arrays.compare(arr1, arr2));
        
        // 流操作
        System.out.println("\n流操作:");
        int sum = Arrays.stream(numbers).sum();
        System.out.println("数字总和: " + sum);
    }
}
```

## 总结

### Arrays工具类的核心优势：

1. **功能全面**：覆盖数组操作的所有常见需求
2. **性能优化**：底层使用高效的算法实现
3. **简洁易用**：静态方法，无需实例化
4. **类型安全**：支持泛型和所有基本类型
5. **线程安全**：所有方法都是无状态的

### 常用方法速查：

| 操作 | 方法                                       | 示例                            |
| ---- | ------------------------------------------ | ------------------------------- |
| 排序 | `sort()`, `parallelSort()`                 | `Arrays.sort(arr)`              |
| 查找 | `binarySearch()`                           | `Arrays.binarySearch(arr, key)` |
| 比较 | `equals()`, `deepEquals()`, `compare()`    | `Arrays.equals(arr1, arr2)`     |
| 填充 | `fill()`, `setAll()`, `parallelSetAll()`   | `Arrays.fill(arr, value)`       |
| 复制 | `copyOf()`, `copyOfRange()`                | `Arrays.copyOf(arr, length)`    |
| 转换 | `toString()`, `deepToString()`, `asList()` | `Arrays.toString(arr)`          |
| 流化 | `stream()`                                 | `Arrays.stream(arr).sum()`      |

Arrays工具类是Java开发者必须掌握的核心工具之一，熟练使用可以大大提高数组处理的效率和代码质量。