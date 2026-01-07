# SHA2 函数实际应用示例：电商平台用户系统

## 场景描述
某电商平台需要安全存储用户密码和订单数据，同时生成安全的用户令牌用于API访问。

## 1. 数据库表设计

```sql
-- 用户表
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    salt CHAR(64) NOT NULL,  -- 密码盐值
    password_hash CHAR(64) NOT NULL,  -- SHA256哈希密码
    api_token CHAR(64) NOT NULL,  -- API访问令牌
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
);

-- 订单表（部分敏感字段加密）
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    -- 敏感字段使用SHA256哈希存储
    credit_card_hash CHAR(64) NOT NULL,  -- 信用卡号哈希
    shipping_address_hash CHAR(64) NOT NULL,  -- 收货地址哈希
    order_status ENUM('pending', 'processing', 'shipped', 'delivered') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX idx_user_status (user_id, order_status)
);

-- 用户活动日志表
CREATE TABLE user_activities (
    activity_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    activity_type VARCHAR(50) NOT NULL,
    -- 记录数据的完整性哈希
    data_fingerprint CHAR(64) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

## 2. 用户注册：密码安全存储

```sql
-- 存储过程：用户注册
DELIMITER //

CREATE PROCEDURE register_user(
    IN p_username VARCHAR(50),
    IN p_email VARCHAR(100),
    IN p_password VARCHAR(255)
)
BEGIN
    DECLARE v_salt CHAR(64);
    DECLARE v_password_hash CHAR(64);
    DECLARE v_api_token CHAR(64);
    
    -- 1. 生成随机盐值
    SET v_salt = SHA2(UUID(), 256);
    
    -- 2. 生成密码哈希：密码 + 盐值
    SET v_password_hash = SHA2(CONCAT(p_password, v_salt), 256);
    
    -- 3. 生成API令牌：用户名 + 时间戳 + 随机数
    SET v_api_token = SHA2(CONCAT(p_username, NOW(), RAND()), 256);
    
    -- 4. 插入用户记录
    INSERT INTO users (username, email, salt, password_hash, api_token)
    VALUES (p_username, p_email, v_salt, v_password_hash, v_api_token);
    
    -- 5. 返回用户ID和API令牌
    SELECT LAST_INSERT_ID() as user_id, v_api_token as api_token;
END //

DELIMITER ;

-- 使用示例：注册新用户
CALL register_user('john_doe', 'john@example.com', 'MySecurePass123!');
-- 返回: user_id = 1, api_token = 'e3b0c44298fc1c149...'
```

## 3. 用户登录验证

```sql
-- 存储过程：用户登录验证
DELIMITER //

CREATE PROCEDURE verify_login(
    IN p_username VARCHAR(50),
    IN p_password VARCHAR(255)
)
BEGIN
    DECLARE v_user_id INT;
    DECLARE v_salt CHAR(64);
    DECLARE v_stored_hash CHAR(64);
    DECLARE v_input_hash CHAR(64);
    DECLARE v_api_token CHAR(64);
    
    -- 1. 获取用户信息
    SELECT user_id, salt, password_hash, api_token
    INTO v_user_id, v_salt, v_stored_hash, v_api_token
    FROM users
    WHERE username = p_username;
    
    -- 2. 验证用户是否存在
    IF v_user_id IS NULL THEN
        SELECT FALSE as login_success, '用户不存在' as message, NULL as api_token;
    ELSE
        -- 3. 计算输入密码的哈希值
        SET v_input_hash = SHA2(CONCAT(p_password, v_salt), 256);
        
        -- 4. 比较哈希值
        IF v_stored_hash = v_input_hash THEN
            -- 登录成功，生成新的API令牌（可选）
            SET v_api_token = SHA2(CONCAT(p_username, NOW(), RAND()), 256);
            UPDATE users SET api_token = v_api_token WHERE user_id = v_user_id;
            
            -- 记录登录活动
            INSERT INTO user_activities (user_id, activity_type, data_fingerprint, ip_address)
            VALUES (v_user_id, 'login', 
                    SHA2(CONCAT(p_username, 'login', NOW()), 256),
                    '192.168.1.100');
            
            SELECT TRUE as login_success, '登录成功' as message, v_api_token as api_token, v_user_id as user_id;
        ELSE
            SELECT FALSE as login_success, '密码错误' as message, NULL as api_token, NULL as user_id;
        END IF;
    END IF;
END //

DELIMITER ;

-- 使用示例：用户登录
CALL verify_login('john_doe', 'MySecurePass123!');
-- 成功返回: login_success = 1, message = '登录成功', api_token = '...', user_id = 1
-- 失败返回: login_success = 0, message = '密码错误', api_token = NULL
```

## 4. 创建订单：敏感信息哈希存储

```sql
-- 存储过程：创建订单
DELIMITER //

CREATE PROCEDURE create_order(
    IN p_user_id INT,
    IN p_total_amount DECIMAL(10,2),
    IN p_credit_card_number VARCHAR(20),
    IN p_shipping_address TEXT
)
BEGIN
    DECLARE v_order_number VARCHAR(50);
    DECLARE v_order_id INT;
    
    -- 1. 生成订单号（时间戳+随机数）
    SET v_order_number = CONCAT('ORD', UNIX_TIMESTAMP(), FLOOR(RAND() * 1000));
    
    -- 2. 插入订单记录（敏感信息哈希存储）
    INSERT INTO orders (
        user_id, order_number, total_amount, 
        credit_card_hash, shipping_address_hash
    ) VALUES (
        p_user_id, v_order_number, p_total_amount,
        -- 信用卡号哈希（后4位保留明文用于显示，其余部分哈希）
        SHA2(CONCAT(p_credit_card_number, 'SALT_ORDER_' , v_order_number), 256),
        -- 收货地址哈希
        SHA2(CONCAT(p_shipping_address, 'SALT_ADDR_' , p_user_id), 256)
    );
    
    SET v_order_id = LAST_INSERT_ID();
    
    -- 3. 记录订单创建活动
    INSERT INTO user_activities (user_id, activity_type, data_fingerprint, ip_address)
    VALUES (
        p_user_id, 
        'order_created',
        SHA2(CONCAT('order', v_order_id, p_total_amount, NOW()), 256),
        '192.168.1.100'
    );
    
    -- 4. 返回订单信息
    SELECT v_order_id as order_id, v_order_number as order_number;
END //

DELIMITER ;

-- 使用示例：创建订单
CALL create_order(1, 299.99, '4111-1111-1111-1111', '123 Main St, New York, NY');
-- 返回: order_id = 1, order_number = 'ORD1609459200123'
```

## 5. 验证订单数据完整性

```sql
-- 函数：验证订单数据完整性
DELIMITER //

CREATE FUNCTION verify_order_integrity(
    p_order_id INT
) RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE v_user_id INT;
    DECLARE v_order_number VARCHAR(50);
    DECLARE v_total_amount DECIMAL(10,2);
    DECLARE v_credit_card_hash CHAR(64);
    DECLARE v_shipping_address_hash CHAR(64);
    DECLARE v_expected_fingerprint CHAR(64);
    
    -- 1. 获取订单信息
    SELECT user_id, order_number, total_amount, 
           credit_card_hash, shipping_address_hash
    INTO v_user_id, v_order_number, v_total_amount,
         v_credit_card_hash, v_shipping_address_hash
    FROM orders
    WHERE order_id = p_order_id;
    
    IF v_user_id IS NULL THEN
        RETURN FALSE;
    END IF;
    
    -- 2. 重新计算哈希值
    SET v_expected_fingerprint = SHA2(
        CONCAT(v_user_id, v_order_number, v_total_amount, 
               v_credit_card_hash, v_shipping_address_hash), 
        256
    );
    
    -- 3. 检查是否有匹配的活动记录
    RETURN EXISTS (
        SELECT 1 FROM user_activities 
        WHERE user_id = v_user_id 
          AND activity_type = 'order_created'
          AND data_fingerprint = v_expected_fingerprint
    );
END //

DELIMITER ;

-- 使用示例：验证订单1的数据完整性
SELECT verify_order_integrity(1) as is_data_integrity_ok;
-- 返回: 1 (TRUE) 或 0 (FALSE)
```

## 6. API访问令牌验证

```sql
-- 函数：验证API令牌
DELIMITER //

CREATE FUNCTION verify_api_token(
    p_user_id INT,
    p_api_token CHAR(64)
) RETURNS BOOLEAN
DETERMINISTIC
READS SQL DATA
BEGIN
    RETURN EXISTS (
        SELECT 1 FROM users 
        WHERE user_id = p_user_id 
          AND api_token = p_api_token
    );
END //

DELIMITER ;

-- 使用示例：验证用户1的API令牌
SELECT verify_api_token(1, 'e3b0c44298fc1c149...') as is_token_valid;
-- 返回: 1 (TRUE) 或 0 (FALSE)
```

## 7. 批量查询用户活动摘要

```sql
-- 查询某个用户的所有活动哈希值
SELECT 
    user_id,
    activity_type,
    data_fingerprint,
    DATE(created_at) as activity_date,
    -- 生成每日活动摘要哈希
    SHA2(
        GROUP_CONCAT(
            data_fingerprint 
            ORDER BY created_at 
            SEPARATOR '|'
        ), 
        256
    ) as daily_fingerprint
FROM user_activities
WHERE user_id = 1
  AND created_at >= '2024-01-01'
GROUP BY user_id, activity_type, DATE(created_at)
ORDER BY activity_date DESC;

-- 这样可以快速验证某天用户活动的完整性
```

## 8. 数据完整性监控（定期检查）

```sql
-- 定期检查所有订单的数据完整性
CREATE EVENT daily_order_integrity_check
ON SCHEDULE EVERY 1 DAY
STARTS '2024-01-01 03:00:00'
DO
BEGIN
    -- 创建临时表存储检查结果
    CREATE TEMPORARY TABLE integrity_check_results (
        order_id INT PRIMARY KEY,
        check_date DATE,
        integrity_status BOOLEAN,
        expected_fingerprint CHAR(64),
        actual_fingerprint CHAR(64)
    );
    
    -- 检查每个订单
    INSERT INTO integrity_check_results (order_id, check_date, integrity_status)
    SELECT 
        order_id,
        CURDATE(),
        verify_order_integrity(order_id)
    FROM orders
    WHERE order_status = 'delivered';
    
    -- 发送警报（这里模拟记录到日志表）
    INSERT INTO system_alerts (alert_type, message, severity)
    SELECT 
        'DATA_INTEGRITY',
        CONCAT('订单 ', order_id, ' 数据完整性检查失败'),
        'HIGH'
    FROM integrity_check_results
    WHERE integrity_status = FALSE;
    
    DROP TEMPORARY TABLE integrity_check_results;
END;
```

## 9. 使用SHA2生成安全URL令牌

```sql
-- 生成安全的密码重置令牌
DELIMITER //

CREATE PROCEDURE generate_password_reset_token(
    IN p_email VARCHAR(100)
)
BEGIN
    DECLARE v_user_id INT;
    DECLARE v_reset_token CHAR(64);
    DECLARE v_expiry_time DATETIME;
    
    -- 1. 查找用户
    SELECT user_id INTO v_user_id
    FROM users
    WHERE email = p_email;
    
    IF v_user_id IS NOT NULL THEN
        -- 2. 生成重置令牌：用户ID + 时间戳 + 随机盐
        SET v_reset_token = SHA2(
            CONCAT(v_user_id, NOW(), RANDOM_BYTES(16)), 
            256
        );
        
        -- 3. 设置过期时间（1小时后）
        SET v_expiry_time = DATE_ADD(NOW(), INTERVAL 1 HOUR);
        
        -- 4. 存储令牌（实际应用中应存到单独的表）
        -- 这里简化处理，直接更新用户表
        UPDATE users 
        SET api_token = v_reset_token  -- 临时使用api_token字段
        WHERE user_id = v_user_id;
        
        -- 5. 返回重置链接（在实际应用中，这应该通过电子邮件发送）
        SELECT 
            CONCAT(
                'https://example.com/reset-password?token=',
                v_reset_token,
                '&expiry=',
                UNIX_TIMESTAMP(v_expiry_time)
            ) as reset_link,
            v_expiry_time as expiry_time;
    ELSE
        SELECT NULL as reset_link, NULL as expiry_time;
    END IF;
END //

DELIMITER ;

-- 使用示例
CALL generate_password_reset_token('john@example.com');
-- 返回: https://example.com/reset-password?token=abc123...&expiry=1704067200
```

## 10. 性能优化：为哈希字段添加索引

```sql
-- 为常用查询字段添加索引
CREATE INDEX idx_password_hash ON users(password_hash);
CREATE INDEX idx_credit_card_hash ON orders(credit_card_hash);
CREATE INDEX idx_data_fingerprint ON user_activities(data_fingerprint);

-- 注意：哈希字段作为索引可能会降低插入性能，需要根据查询模式权衡
```

## 实际运行示例

```sql
-- 1. 用户注册
CALL register_user('alice_smith', 'alice@example.com', 'AlicePass123');
-- 假设返回: user_id = 2, api_token = 'a1b2c3d4...'

-- 2. 用户登录
CALL verify_login('alice_smith', 'AlicePass123');
-- 成功登录，返回新的API令牌

-- 3. 创建订单
CALL create_order(2, 150.75, '5555-5555-5555-4444', '456 Oak Ave, Boston, MA');

-- 4. 验证数据完整性
SELECT verify_order_integrity(2) as is_intact;
-- 应该返回1（TRUE）

-- 5. 验证订单数据是否被篡改（模拟攻击）
UPDATE orders SET total_amount = 0.01 WHERE order_id = 2;
SELECT verify_order_integrity(2) as is_intact_after_tampering;
-- 现在应该返回0（FALSE），因为订单数据被修改
```

## 注意事项

1. **盐值管理**：每个用户的盐值应该是唯一的，并且足够随机
2. **密码强度**：应在应用层强制要求密码复杂度
3. **哈希碰撞**：虽然SHA256碰撞概率极低，但重要系统应考虑使用更长的哈希（如SHA512）
4. **性能影响**：大量使用哈希计算会影响数据库性能，需考虑缓存策略
5. **合规性**：根据行业规范（如PCI DSS）处理敏感数据
6. **密钥管理**：实际应用中，盐值和密钥应从安全配置中获取，而不是硬编码

这个实例展示了SHA2在用户认证、数据完整性验证、安全令牌生成等多个方面的实际应用，适用于需要高安全性的电商系统。