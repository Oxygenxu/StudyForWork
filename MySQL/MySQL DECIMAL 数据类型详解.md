# MySQL DECIMAL 数据类型详解

## 基本概念

**DECIMAL** 是 MySQL 中用于存储**精确数值**的定点数数据类型，特别适合存储财务数据、货币金额等需要精确计算的场景。

## 基本语法

```sql
DECIMAL(M, D)
```
或
```sql
DECIMAL(P, S)  -- P 是精度，S 是标度
```

## 参数说明

| 参数  | 含义 | 范围           | 说明                      |
| ----- | ---- | -------------- | ------------------------- |
| **M** | 精度 | 1-65           | 总位数（整数位 + 小数位） |
| **D** | 标度 | 0-30，且 D ≤ M | 小数位数                  |

## 存储示例

```sql
-- 创建使用 DECIMAL 的表
CREATE TABLE financial_data (
    id INT PRIMARY KEY,
    price DECIMAL(10, 2),      -- 总共10位，2位小数
    tax_rate DECIMAL(5, 4),    -- 总共5位，4位小数
    total_amount DECIMAL(20, 2) -- 大金额，2位小数
);

-- 插入数据
INSERT INTO financial_data VALUES
(1, 999999.99, 0.0825, 1234567890123456.78),
(2, 123.45, 0.1500, 987654321098.76);
```

## 存储范围示例

```sql
DECIMAL(5, 2)  -- 存储范围：-999.99 到 999.99
DECIMAL(10, 0) -- 存储范围：-9999999999 到 9999999999（纯整数）
DECIMAL(6, 4)  -- 存储范围：-99.9999 到 99.9999
```

## 与 FLOAT/DOUBLE 的对比

| 特性         | DECIMAL                | FLOAT/DOUBLE               |
| ------------ | ---------------------- | -------------------------- |
| **精度**     | 精确存储               | 近似存储                   |
| **适用场景** | 财务计算、货币         | 科学计算、不需要精确的场景 |
| **存储方式** | 字符串形式存储         | 二进制浮点数               |
| **存储空间** | 可变，每4字节存9位数字 | 固定（4或8字节）           |
| **精度问题** | 无精度损失             | 可能有舍入误差             |

### 精度问题示例
```sql
-- DECIMAL 精确计算
SELECT 0.1 + 0.2;                    -- 返回 0.3（精确）
SELECT CAST(0.1 AS DECIMAL(10,2)) + 
       CAST(0.2 AS DECIMAL(10,2));   -- 返回 0.30（精确）

-- FLOAT 可能产生误差
SELECT CAST(0.1 AS FLOAT) + 
       CAST(0.2 AS FLOAT);           -- 可能返回 0.30000001192092896
```

## DECIMAL 的存储空间

DECIMAL 的存储空间根据精度 M 动态分配：

| 精度范围     | 存储空间（字节）   |
| ------------ | ------------------ |
| 0-9 位数字   | 4 字节             |
| 10-18 位数字 | 8 字节             |
| 19-28 位数字 | 12 字节            |
| 29-38 位数字 | 16 字节            |
| 39-65 位数字 | 每9位数字增加4字节 |

### 存储空间计算示例
```sql
DECIMAL(10,2)  -- 10位数字，需要 4+1=5字节（实际按8字节对齐）
DECIMAL(20,2)  -- 20位数字，需要 12字节
DECIMAL(38,10) -- 38位数字，需要 17字节
```

## 实际应用场景

### 场景1：财务系统
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    amount DECIMAL(15, 2),        -- 订单金额
    tax DECIMAL(15, 2),           -- 税额
    discount DECIMAL(5, 4),       -- 折扣率（0.0000-0.9999）
    final_amount DECIMAL(15, 2)   -- 最终金额
);
```

### 场景2：银行系统
```sql
CREATE TABLE accounts (
    account_no VARCHAR(20),
    balance DECIMAL(20, 2),       -- 账户余额
    interest_rate DECIMAL(7, 6),  -- 利率（0.000001-9.999999%）
    credit_limit DECIMAL(15, 2)   -- 信用额度
);
```

### 场景3：电商系统
```sql
CREATE TABLE products (
    product_id INT,
    price DECIMAL(10, 2),         -- 单价
    cost DECIMAL(10, 2),          -- 成本
    profit_margin DECIMAL(5, 4),  -- 利润率
    stock DECIMAL(12, 3)          -- 库存（支持小数，如 0.5kg）
);
```

## 运算和函数

### 基本运算
```sql
-- 加减乘除
SELECT price * quantity AS total FROM sales;  -- 自动使用 DECIMAL 计算

-- 注意：除法可能需要调整精度
SELECT amount / 100 AS percentage FROM data;

-- 使用 ROUND 函数
SELECT ROUND(price, 2) FROM products;        -- 四舍五入到2位小数

-- 使用 TRUNCATE 函数
SELECT TRUNCATE(price, 2) FROM products;     -- 截断到2位小数
```

### 精度控制示例
```sql
-- 创建表时定义精度
CREATE TABLE calculations (
    a DECIMAL(10, 4),
    b DECIMAL(10, 4),
    result DECIMAL(11, 4)  -- 注意：乘法可能需要更大的精度
);

-- 计算时注意精度溢出
SELECT 
    a,
    b,
    a * b AS multiplication,           -- 可能精度溢出
    CAST(a * b AS DECIMAL(20, 8)) AS safe_multiplication
FROM calculations;
```

## 最佳实践

### 1. 合理选择精度
```sql
-- 合适的精度选择
DECIMAL(10, 2)  -- 一般金额：最大 99,999,999.99
DECIMAL(15, 2)  -- 较大金额：最大 9,999,999,999,999.99
DECIMAL(20, 8)  -- 加密货币：需要高小数精度
```

### 2. 避免不必要的精度
```sql
-- 不推荐：过度精度浪费空间
price DECIMAL(38, 10)  -- 对于普通商品价格，精度过高

-- 推荐：根据实际需求
price DECIMAL(10, 2)   -- 足够存储 99999999.99
```

### 3. 计算时的精度管理
```sql
-- 使用 CAST 控制计算精度
SELECT 
    price,
    quantity,
    CAST(price * quantity AS DECIMAL(15, 2)) AS total
FROM sales;

-- 或者创建计算列
ALTER TABLE sales 
ADD COLUMN total DECIMAL(15, 2) 
GENERATED ALWAYS AS (price * quantity) STORED;
```

## 常见问题与解决方案

### 问题1：精度溢出
```sql
-- 错误：Out of range value for column
-- 原因：插入的值超出 DECIMAL(M, D) 的范围
INSERT INTO products (price) VALUES (100000.00);  -- DECIMAL(5,2) 最大 999.99

-- 解决方案：扩大精度
ALTER TABLE products MODIFY price DECIMAL(10, 2);
```

### 问题2：四舍五入规则
```sql
-- DECIMAL 使用"银行家舍入法"（四舍六入五成双）
SELECT ROUND(2.5);  -- 返回 2
SELECT ROUND(3.5);  -- 返回 4

-- 如果需要传统四舍五入
SELECT FORMAT(2.5, 0);  -- 返回 3
```

### 问题3：与整数运算
```sql
-- DECIMAL 与整数运算时，结果仍是 DECIMAL
SELECT price * 100 FROM products;  -- 结果精度可能变化

-- 明确转换类型
SELECT CAST(price AS DECIMAL(12,2)) * 100 FROM products;
```

## 性能考虑

1. **存储空间**：DECIMAL 比 FLOAT/DOUBLE 占用更多空间
2. **计算速度**：DECIMAL 运算比浮点数慢
3. **索引使用**：DECIMAL 列可以创建索引，但比整数索引稍慢
4. **内存占用**：查询大量 DECIMAL 数据时占用更多内存

## 版本差异

### MySQL 5.7 及之前
```sql
-- 默认 DECIMAL 精度
DECIMAL(10, 0)  -- 默认精度
```

### MySQL 8.0 及之后
```sql
-- 支持更灵活的精度设置
-- 更好的优化器处理
```

## 总结

**DECIMAL 关键要点：**

1. **精确存储**：无精度损失，适合财务计算
2. **参数设置**：DECIMAL(M, D)，M 是总位数，D 是小数位数
3. **存储格式**：以字符串形式存储，保证精度
4. **应用场景**：货币金额、税率、百分比等需要精确计算的场景
5. **性能权衡**：精度越高，占用空间越大，计算越慢

**使用建议：**
- 财务数据必须使用 DECIMAL
- 根据实际需要选择最小够用的精度
- 注意计算过程中的精度溢出问题
- 了解四舍五入规则，必要时使用 ROUND() 或 FORMAT()