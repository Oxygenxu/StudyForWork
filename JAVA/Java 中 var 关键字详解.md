# Java 中 var 关键字详解

`var` 是 Java 10 引入的**局部变量类型推断**关键字，它让编译器根据初始化表达式自动推断变量类型。

## 1. **基本语法和特性**

### 1.1 基本语法
```java
// 传统方式
String message = "Hello, World!";
List<String> names = new ArrayList<>();

// 使用 var
var message = "Hello, World!";          // 编译器推断为 String
var names = new ArrayList<String>();    // 编译器推断为 ArrayList<String>
```

### 1.2 关键特性
- **类型推断**：编译器根据右侧表达式推断类型
- **编译时特性**：类型检查在编译时完成，运行时与显式类型相同
- **只读类型**：一旦推断，类型固定，不能改变
- **局部变量**：只能在方法、构造函数、局部代码块中使用

## 2. **使用场景和示例**

### 2.1 减少冗余类型声明
```java
// 场景1：减少泛型冗余
// 传统方式
Map<String, List<Map<String, Object>>> complexMap = new HashMap<String, List<Map<String, Object>>>();

// 使用 var（更简洁）
var complexMap = new HashMap<String, List<Map<String, Object>>>();

// 场景2：链式调用
var result = getData()
    .stream()
    .filter(item -> item.isActive())
    .map(item -> item.getName())
    .collect(Collectors.toList());
```

### 2.2 增强代码可读性
```java
import java.util.*;

public class VarReadability {
    public static void main(String[] args) {
        // 场景1：迭代器
        var iterator = getUsers().iterator();
        while (iterator.hasNext()) {
            var user = iterator.next();
            processUser(user);
        }
        
        // 场景2：for-each 循环
        for (var entry : getMap().entrySet()) {
            var key = entry.getKey();
            var value = entry.getValue();
            System.out.println(key + ": " + value);
        }
        
        // 场景3：传统 for 循环
        for (var i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
    
    static Map<String, Integer> getMap() {
        return Map.of("A", 1, "B", 2, "C", 3);
    }
}
```

### 2.3 匿名类和局部类
```java
public class AnonymousClassExample {
    public static void main(String[] args) {
        // 传统方式
        Runnable runnable1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Running...");
            }
        };
        
        // 使用 var（更简洁）
        var runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Running with var...");
            }
        };
        
        new Thread(runnable2).start();
        
        // 局部类
        var local = new Object() {
            String name = "John";
            int age = 30;
            
            void print() {
                System.out.println(name + " is " + age + " years old");
            }
        };
        
        local.print();
    }
}
```

## 3. **限制和约束**

### 3.1 不能使用 var 的场景
```java
public class VarLimitations {
    // 1. 不能用于字段（成员变量）
    // private var name = "John";  // 编译错误
    
    // 2. 不能用于方法参数
    // public void process(var data) { }  // 编译错误
    
    // 3. 不能用于方法返回类型
    // public var getResult() { return 42; }  // 编译错误
    
    // 4. 不能用于 catch 块参数
    public void test() {
        try {
            throw new IOException();
        } catch (IOException e) {  // 不能用 var e
            e.printStackTrace();
        }
    }
    
    // 5. 不能用于 lambda 参数类型
    public void lambdaTest() {
        // var f = (var x, var y) -> x + y;  // 编译错误（Java 11 已允许）
        
        // Java 11+ 允许在 lambda 参数中使用 var
        // var f = (var x, var y) -> x + y;  // Java 11+ 可以
    }
}
```

### 3.2 必须初始化的约束
```java
public class VarInitialization {
    public static void main(String[] args) {
        // 1. 必须初始化
        var name;  // 编译错误：不能使用 'var' 而不带初始化程序
        name = "John";
        
        // 2. 不能初始化为 null（无法推断类型）
        var data = null;  // 编译错误：无法推断类型
        
        // 3. 可以稍后赋值，但必须有初始化
        var count = 0;  // 推断为 int
        count = 10;     // 可以重新赋值，但类型必须是 int
        // count = "ten";  // 编译错误：String 不能转换为 int
    }
}
```

### 3.3 数组初始化限制
```java
public class VarArrays {
    public static void main(String[] args) {
        // 传统方式
        int[] numbers = {1, 2, 3};
        String[] names = new String[5];
        
        // 使用 var 的限制
        var numbers2 = new int[]{1, 2, 3};  // 正确
        // var numbers3 = {1, 2, 3};         // 错误：不能使用数组初始化器
        
        var names2 = new String[5];         // 正确
        names2[0] = "John";
        
        // 多维数组
        var matrix = new int[3][3];         // 正确：int[][]
    }
}
```

## 4. **最佳实践和指导原则**

### 4.1 何时使用 var
```java
public class WhenToUseVar {
    // 场景1：类型很明显
    public void obviousTypes() {
        var list = new ArrayList<String>();       // 推荐：类型明显
        var map = new HashMap<String, Integer>(); // 推荐：类型明显
        var message = "Hello";                   // 推荐：明显是 String
    }
    
    // 场景2：减少冗余
    public void reduceVerbosity() {
        // 传统方式
        SomeVeryLongClassName<AnotherLongType> obj = 
            new SomeVeryLongClassName<AnotherLongType>();
            
        // 使用 var（更简洁）
        var obj = new SomeVeryLongClassName<AnotherLongType>();
    }
    
    // 场景3：中间变量
    public void intermediateVariables() {
        // 临时变量，类型不重要
        var temp = calculateSomething();
        process(temp);
    }
}
```

### 4.2 何时避免使用 var
```java
public class WhenToAvoidVar {
    // 场景1：类型不明显时
    public void unclearTypes() {
        // 不推荐：无法一眼看出类型
        var data = getData();  // data 是什么类型？
        
        // 推荐：明确类型
        UserData data = getData();  // 明确是 UserData
    }
    
    // 场景2：重要的 API 返回类型
    public void importantAPITypes() {
        // 不推荐：重要的返回类型应该明确
        var result = executeImportantOperation();
        
        // 推荐：明确类型，提高可读性
        OperationResult result = executeImportantOperation();
    }
    
    // 场景3：数字类型
    public void numericTypes() {
        // 小心：可能不是预期的类型
        var count = 1;     // int
        var price = 9.99;  // double
        var big = 100L;    // long
        
        // 明确指定类型更安全
        long totalCount = 1L;
        BigDecimal price2 = BigDecimal.valueOf(9.99);
    }
}
```

## 5. **与动态类型的区别**

```java
public class StaticVsDynamic {
    public static void main(String[] args) {
        // Java 的 var 是静态类型
        var name = "John";   // 编译时推断为 String
        // name = 123;       // 编译错误：不能将 int 赋给 String
        
        // 对比 JavaScript（动态类型）
        // let name = "John";  // 运行时可以是任意类型
        // name = 123;         // 允许：运行时改变类型
    }
}
```

## 6. **与其他语言的比较**

```java
// Java 的 var 类似于：
// C#: var name = "John";
// C++: auto name = "John";
// TypeScript: let name = "John";

// 但不同于：
// JavaScript: let/var name = "John";  // 动态类型
// Python: name = "John"              // 动态类型
```

## 7. **实际项目中的使用示例**

### 7.1 Java 11+ 的 HTTP Client
```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class HttpClientExample {
    public static void main(String[] args) throws Exception {
        // 使用 var 简化 HTTP Client 代码
        var client = HttpClient.newHttpClient();
        var request = HttpRequest.newBuilder()
                .uri(URI.create("https://api.example.com/data"))
                .build();
        
        var response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
        
        System.out.println("Status: " + response.statusCode());
        System.out.println("Body: " + response.body());
    }
}
```

### 7.2 Stream API 处理
```java
import java.util.*;
import java.util.stream.Collectors;

public class StreamExample {
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 25),
            new Person("Bob", 30),
            new Person("Charlie", 35)
        );
        
        // 使用 var 简化流操作
        var result = people.stream()
            .filter(p -> p.getAge() > 25)
            .map(Person::getName)
            .collect(Collectors.toList());
        
        System.out.println("Result: " + result);
        
        // 分组操作
        var byFirstLetter = people.stream()
            .collect(Collectors.groupingBy(
                p -> p.getName().substring(0, 1)
            ));
        
        byFirstLetter.forEach((key, value) -> 
            System.out.println(key + ": " + value));
    }
}

class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

### 7.3 文件操作
```java
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class FileOperations {
    public static void main(String[] args) throws IOException {
        // 读取文件
        var path = Paths.get("data.txt");
        var lines = Files.readAllLines(path);
        
        // 处理文件内容
        var wordCount = lines.stream()
            .flatMap(line -> Arrays.stream(line.split("\\s+")))
            .filter(word -> !word.isEmpty())
            .count();
        
        System.out.println("Total words: " + wordCount);
        
        // 写入文件
        var output = List.of("Line 1", "Line 2", "Line 3");
        var outputPath = Paths.get("output.txt");
        Files.write(outputPath, output);
    }
}
```

## 8. **性能影响**

```java
public class VarPerformance {
    public static void main(String[] args) {
        // var 是编译时特性，对运行时性能无影响
        // 以下两段代码编译后完全相同
        
        // 使用 var
        var start1 = System.currentTimeMillis();
        for (var i = 0; i < 1000000; i++) {
            var temp = i * 2;
        }
        var end1 = System.currentTimeMillis();
        
        // 不使用 var
        long start2 = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            int temp = i * 2;
        }
        long end2 = System.currentTimeMillis();
        
        System.out.println("Var time: " + (end1 - start1) + "ms");
        System.out.println("Explicit time: " + (end2 - start2) + "ms");
        // 两者性能相同
    }
}
```

## 9. **团队编码规范建议**

```java
/**
 * 公司/团队编码规范示例
 */
public class CodingGuidelines {
    
    // 1. 对于明显类型，使用 var
    public void guideline1() {
        var list = new ArrayList<String>();      // ✓ 推荐
        var map = new HashMap<Integer, String>(); // ✓ 推荐
        var message = "Hello";                   // ✓ 推荐
    }
    
    // 2. 对于重要类型，显示声明
    public void guideline2() {
        // ✗ 避免：返回类型不明显
        var result = calculateImportantThing();
        
        // ✓ 推荐：明确类型
        CalculationResult result = calculateImportantThing();
    }
    
    // 3. 保持一致性
    public void guideline3() {
        // 在同一个方法中保持一致的风格
        var names = new ArrayList<String>();     // ✓
        List<String> addresses = new ArrayList<>(); // ✗ 不一致
        
        // 应该统一使用 var 或统一使用显式类型
    }
    
    // 4. 为 var 变量选择有意义的名称
    public void guideline4() {
        // ✗ 不好：变量名没有提供类型信息
        var x = getUsers();
        
        // ✓ 好：变量名暗示了类型
        var userList = getUsers();
        var userMap = getUserMap();
    }
}
```

## 总结

### **var 的关键要点：**

1. **作用**：局部变量类型推断
2. **范围**：只能用于局部变量（方法内、代码块内）
3. **要求**：必须初始化，不能为 null
4. **时机**：编译时推断，运行时类型固定
5. **性能**：与显式类型完全相同

### **使用建议：**

- ✅ **适合使用 var**：
  - 类型明显时（`new ArrayList<String>()`）
  - 减少冗余代码时（复杂的泛型声明）
  - 中间临时变量
  - 增强循环可读性时

- ❌ **避免使用 var**：
  - 类型不明确时
  - 重要的 API 返回类型
  - 基本数值类型需要明确时（如 long 和 int）
  - 会降低代码可读性时

`var` 是一个便利的特性，旨在减少样板代码，但不应牺牲代码的清晰度和可读性。合理使用 `var` 可以让代码更简洁，过度使用则可能让代码更难理解。