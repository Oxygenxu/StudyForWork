## MySQL 唯一性约束简介

**唯一性约束（UNIQUE Constraint）** 用于确保表中的某一列或多列的值都是唯一的，不允许出现重复值。

### 主要特性：

1. **唯一性保证**
   - 约束列的值在表中必须是唯一的
   - 可以有多个 NULL 值（除非同时有 NOT NULL 约束）

2. **作用范围**
   - 可以作用于单列或多列（组合唯一性）
   - 一张表可以有多个唯一约束

### 创建方式：

#### 1. **建表时创建**
```sql
-- 单列唯一约束
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,  -- 列级约束
    username VARCHAR(50) UNIQUE
);

-- 多列组合唯一约束
CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    UNIQUE (user_id, role_id)  -- 表级约束
);
```

#### 2. **修改表时添加**
```sql
-- 添加单列唯一约束
ALTER TABLE users ADD UNIQUE (email);

-- 添加多列组合唯一约束
ALTER TABLE user_roles ADD UNIQUE (user_id, role_id);

-- 命名唯一约束
ALTER TABLE users ADD CONSTRAINT uk_users_email UNIQUE (email);
```

### 使用示例：

```sql
-- 创建带唯一约束的表
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_code VARCHAR(20) UNIQUE,
    name VARCHAR(100),
    sku VARCHAR(50),
    UNIQUE KEY (sku)  -- 另一种写法
);

-- 插入数据
INSERT INTO products (product_code, name, sku) 
VALUES ('P001', 'Laptop', 'SKU001');  -- 成功

INSERT INTO products (product_code, name, sku) 
VALUES ('P002', 'Phone', 'SKU001');  -- 失败：sku重复

-- 允许NULL值
INSERT INTO products (product_code, name, sku) 
VALUES ('P003', 'Tablet', NULL);  -- 成功

INSERT INTO products (product_code, name, sku) 
VALUES ('P004', 'Monitor', NULL);  -- 成功（允许多个NULL）
```

### 组合唯一约束示例：

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    semester VARCHAR(10),
    UNIQUE (student_id, course_id, semester)  -- 三列组合必须唯一
);

-- 允许的插入
INSERT INTO enrollments VALUES (1, 101, 'Fall2023');
INSERT INTO enrollments VALUES (1, 102, 'Fall2023');  -- 不同课程
INSERT INTO enrollments VALUES (1, 101, 'Spring2024');  -- 不同学期

-- 不允许的插入
INSERT INTO enrollments VALUES (1, 101, 'Fall2023');  -- 完全重复
```

### 唯一约束与主键的区别：

| 特性 | 主键 (PRIMARY KEY) | 唯一约束 (UNIQUE)     |
| ---- | ------------------ | --------------------- |
| 空值 | 不允许 NULL        | 允许 NULL（允许多个） |
| 数量 | 每表只能有一个     | 每表可以有多个        |
| 索引 | 自动创建聚集索引   | 自动创建非聚集索引    |
| 用途 | 标识行             | 确保数据唯一性        |

### 查看和管理约束：

```sql
-- 查看表的约束信息
SHOW CREATE TABLE users;

-- 查看索引（唯一约束会创建唯一索引）
SHOW INDEX FROM users;

-- 删除唯一约束（通过删除索引实现）
ALTER TABLE users DROP INDEX email;

-- 删除命名约束
ALTER TABLE users DROP CONSTRAINT uk_users_email;
```

### 使用场景：

1. **防止重复数据**
   - 用户邮箱、手机号
   - 身份证号
   - 产品编号

2. **组合业务规则**
   - 学生选课（同一学生同一课程同一学期只能选一次）
   - 员工排班（同一员工同一时间段不能重复排班）

3. **替代非必要的主键**
   - 当已有主键，但其他列也需要保证唯一性时

### 注意事项：

1. 唯一约束会自动创建唯一索引
2. 插入或更新数据时会检查唯一性，影响性能
3. 组合唯一约束中，所有列值完全相同时才会违反约束
4. 在允许NULL的列上，多个NULL值不会违反唯一约束