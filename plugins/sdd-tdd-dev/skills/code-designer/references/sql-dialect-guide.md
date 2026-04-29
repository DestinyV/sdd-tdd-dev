# SQL 方言对照指南

> code-designer 阶段 2.5.2 数据库方言检测参考

## 方言检测优先级

1. 读取项目现有迁移文件（如 `.sql` 中使用 `SERIAL` → PostgreSQL）
2. 读取项目配置文件（如 `ormconfig.json`、`database.yml` 等）
3. 无法检测时默认使用 **MySQL**

---

## 数据类型差异

| 概念 | MySQL | PostgreSQL |
|------|-------|------------|
| 自增主键 | `BIGINT PRIMARY KEY AUTO_INCREMENT` | `BIGSERIAL PRIMARY KEY` |
| 布尔值 | `TINYINT(1)` | `BOOLEAN` |
| UUID | `VARCHAR(36)` | `UUID` |
| 当前时间 | `DEFAULT CURRENT_TIMESTAMP` | `DEFAULT NOW()` |
| 更新自动时间 | `ON UPDATE CURRENT_TIMESTAMP` | 需触发器或应用层处理 |
| 注释 | `COMMENT '说明'` | `COMMENT ON COLUMN table.column IS '说明'` |
| 长文本 | `TEXT` | `TEXT` |
| JSON | `JSON` | `JSONB`（推荐） |
| 枚举 | `ENUM('a','b','c')` | `CREATE TYPE` + 列引用 |
| 数组 | 不支持 | `TEXT[]`, `INT[]` |
| 自增起始 | `AUTO_INCREMENT = 1000` | `ALTER SEQUENCE ... START WITH` |

---

## DDL 语法差异

### CREATE TABLE

**MySQL**:
```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
  name VARCHAR(100) NOT NULL COMMENT '用户名',
  email VARCHAR(255) UNIQUE NOT NULL COMMENT '邮箱',
  status TINYINT(1) DEFAULT 1 COMMENT '状态: 0=禁用 1=启用',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

**PostgreSQL**:
```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  status SMALLINT DEFAULT 1,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

COMMENT ON TABLE users IS '用户表';
COMMENT ON COLUMN users.id IS '用户ID';
COMMENT ON COLUMN users.name IS '用户名';
COMMENT ON COLUMN users.email IS '邮箱';
COMMENT ON COLUMN users.status IS '状态: 0=禁用 1=启用';
```

### ALTER TABLE

**MySQL**:
```sql
ALTER TABLE users ADD COLUMN age INT COMMENT '年龄';
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users MODIFY COLUMN name VARCHAR(200) COMMENT '用户名';
```

**PostgreSQL**:
```sql
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users ALTER COLUMN name TYPE VARCHAR(200);

COMMENT ON COLUMN users.age IS '年龄';
```

---

## 索引语法差异

### 普通索引

**MySQL**:
```sql
CREATE INDEX idx_users_email ON users(email);
```

**PostgreSQL**:
```sql
CREATE INDEX idx_users_email ON users(email);
-- 支持条件索引
CREATE INDEX idx_users_active ON users(email) WHERE status = 1;
```

### 复合索引

**MySQL**:
```sql
CREATE INDEX idx_users_status_name ON users(status, name);
```

**PostgreSQL**:
```sql
CREATE INDEX idx_users_status_name ON users(status, name);
-- 支持降序索引
CREATE INDEX idx_users_created_desc ON users(created_at DESC);
```

### 唯一索引

**MySQL**:
```sql
ALTER TABLE users ADD UNIQUE INDEX uk_users_email (email);
```

**PostgreSQL**:
```sql
ALTER TABLE users ADD CONSTRAINT uk_users_email UNIQUE (email);
```

---

## 约束语法差异

### 外键

**MySQL**:
```sql
ALTER TABLE orders ADD CONSTRAINT fk_orders_user_id
  FOREIGN KEY (user_id) REFERENCES users(id)
  ON DELETE CASCADE ON UPDATE CASCADE;
```

**PostgreSQL**:
```sql
ALTER TABLE orders ADD CONSTRAINT fk_orders_user_id
  FOREIGN KEY (user_id) REFERENCES users(id)
  ON DELETE CASCADE ON UPDATE CASCADE;
```

### 检查约束

**MySQL** (8.0.16+):
```sql
ALTER TABLE orders ADD CONSTRAINT chk_orders_amount
  CHECK (amount > 0);
```

**PostgreSQL**:
```sql
ALTER TABLE orders ADD CONSTRAINT chk_orders_amount
  CHECK (amount > 0);
```

---

## 回滚脚本格式

### DDL 回滚

**MySQL & PostgreSQL 通用**:
```sql
-- 回滚: DROP TABLE IF EXISTS users;
-- 回滚: ALTER TABLE users DROP COLUMN age;
-- 回滚: DROP INDEX idx_users_email ON users;  -- MySQL
-- 回滚: DROP INDEX idx_users_email;            -- PostgreSQL
```

---

## 方言特定扩展

### MySQL 特有

- `ENGINE=InnoDB` — 存储引擎
- `DEFAULT CHARSET=utf8mb4` — 字符集
- `ROW_FORMAT=DYNAMIC` — 行格式
- `FULLTEXT INDEX` — 全文索引

### PostgreSQL 特有

- `JSONB` — 二进制 JSON（支持索引）
- `ARRAY` — 数组类型
- `PARTITION BY` — 表分区
- `MATERIALIZED VIEW` — 物化视图
- `WITH (fillfactor=90)` — 存储参数
