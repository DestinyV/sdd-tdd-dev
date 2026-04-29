# SQL 文件生成模板

> code-designer 阶段 2.5.2 使用此模板生成独立可执行 SQL 文件

## 输出目录

```
spec-dev/{requirement_desc_abstract}/design/sql/
```

## 文件清单

| 文件名 | 内容 | 触发条件 |
|--------|------|---------|
| `V{version}__create_tables.sql` | 新建表的完整 DDL | 有新表时 |
| `V{version}__alter_tables.sql` | 修改现有表的 DDL | 有表结构变更时 |
| `V{version}__init_data.sql` | 初始数据 DML | 需要初始化/迁移数据时 |
| `migration-plan.md` | 迁移执行计划 | 始终生成 |

> version 默认从 `V1` 开始。如项目已有迁移文件（如 `V3__xxx.sql`），则取最大值+1。

---

## 通用约定

### 命名规范
- 表名：`snake_case`，小写，使用业务语义明确的名称
- 列名：`snake_case`，小写
- 索引名：`idx_{table}_{column}` 或 `idx_{table}_{col1}_{col2}`
- 唯一约束名：`uk_{table}_{column}`
- 外键名：`fk_{table}_{column}`

### 审计字段（所有表必须包含）
- `id` — 主键
- `created_at` — 创建时间
- `updated_at` — 更新时间
- 逻辑删除场景：`deleted_at`

### 回滚脚本
- 每个 DDL/DML 语句后必须附带回滚注释
- 回滚注释格式：`-- ROLLBACK: <回滚语句>`

---

## V{version}__create_tables.sql 格式

```sql
-- ============================================================
-- 迁移脚本: V{version}__create_tables.sql
-- 描述: [本次迁移的简要描述]
-- 日期: YYYY-MM-DD
-- 数据库方言: [mysql | postgresql]
-- ============================================================

-- 开始事务
START TRANSACTION;  -- MySQL / PostgreSQL
-- BEGIN;            -- PostgreSQL 也可用 BEGIN

-- ---------------------------------------------------------------
-- 表: [table_name]
-- 描述: [表的主要用途]
-- ---------------------------------------------------------------
CREATE TABLE [table_name] (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,  -- MySQL: AUTO_INCREMENT
  -- id BIGSERIAL PRIMARY KEY,           -- PostgreSQL: SERIAL
  [column1] [type] [constraints] COMMENT '[说明]',
  [column2] [type] [constraints] COMMENT '[说明]',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  -- created_at TIMESTAMP DEFAULT NOW(),  -- PostgreSQL
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  -- updated_at TIMESTAMP DEFAULT NOW(),  -- PostgreSQL (需触发器更新)
  deleted_at DATETIME DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='[表说明]';
-- ROLLBACK: DROP TABLE IF EXISTS [table_name];

-- 索引: [table_name].[column]
CREATE INDEX idx_[table]_[column] ON [table_name]([column]);
-- ROLLBACK: DROP INDEX idx_[table]_[column] ON [table_name];  -- MySQL
-- ROLLBACK: DROP INDEX idx_[table]_[column];                  -- PostgreSQL

-- 唯一约束: [table_name].[column]
ALTER TABLE [table_name] ADD CONSTRAINT uk_[table]_[column] UNIQUE ([column]);
-- ROLLBACK: ALTER TABLE [table_name] DROP INDEX uk_[table]_[column];  -- MySQL
-- ROLLBACK: ALTER TABLE [table_name] DROP CONSTRAINT uk_[table]_[column];  -- PostgreSQL

-- 外键: [table_name].[column] → [ref_table].[ref_column]
ALTER TABLE [table_name] ADD CONSTRAINT fk_[table]_[column]
  FOREIGN KEY ([column]) REFERENCES [ref_table]([ref_column])
  ON DELETE CASCADE ON UPDATE CASCADE;
-- ROLLBACK: ALTER TABLE [table_name] DROP FOREIGN KEY fk_[table]_[column];  -- MySQL
-- ROLLBACK: ALTER TABLE [table_name] DROP CONSTRAINT fk_[table]_[column];   -- PostgreSQL

-- ---------------------------------------------------------------
-- 提交事务
-- ---------------------------------------------------------------
COMMIT;
```

---

## V{version}__alter_tables.sql 格式

```sql
-- ============================================================
-- 迁移脚本: V{version}__alter_tables.sql
-- 描述: [本次表结构变更的简要描述]
-- 日期: YYYY-MM-DD
-- 数据库方言: [mysql | postgresql]
-- ============================================================

START TRANSACTION;

-- 添加列: [table_name].[column]
ALTER TABLE [table_name] ADD COLUMN [column] [type] [constraints] COMMENT '[说明]';
-- ROLLBACK: ALTER TABLE [table_name] DROP COLUMN [column];

-- 修改列: [table_name].[column]
ALTER TABLE [table_name] MODIFY COLUMN [column] [new_type] [new_constraints] COMMENT '[说明]';
-- ROLLBACK: ALTER TABLE [table_name] MODIFY COLUMN [column] [old_type] [old_constraints] COMMENT '[说明]';
-- PostgreSQL: ALTER TABLE [table_name] ALTER COLUMN [column] TYPE [new_type];

-- 添加索引
ALTER TABLE [table_name] ADD INDEX idx_[table]_[column] ([column]);
-- ROLLBACK: ALTER TABLE [table_name] DROP INDEX idx_[table]_[column];

-- 添加列默认值
ALTER TABLE [table_name] ALTER COLUMN [column] SET DEFAULT [value];
-- ROLLBACK: ALTER TABLE [table_name] ALTER COLUMN [column] DROP DEFAULT;

COMMIT;
```

---

## V{version}__init_data.sql 格式

```sql
-- ============================================================
-- 数据脚本: V{version}__init_data.sql
-- 描述: [初始数据/数据迁移的简要描述]
-- 日期: YYYY-MM-DD
-- 数据库方言: [mysql | postgresql]
-- ============================================================

START TRANSACTION;

-- 插入初始数据: [table_name]
INSERT INTO [table_name] ([column1], [column2], [created_at], [updated_at])
VALUES
  ('value1', 'value2', NOW(), NOW()),
  ('value3', 'value4', NOW(), NOW());
-- ROLLBACK: DELETE FROM [table_name] WHERE [column1] IN ('value1', 'value3');

-- 数据迁移: [description]
UPDATE [table_name] SET [column] = [expression] WHERE [condition];
-- ROLLBACK: UPDATE [table_name] SET [column] = [original_value] WHERE [condition];

COMMIT;
```

---

## migration-plan.md 格式

```markdown
# 数据库迁移计划

**需求**: [requirement_desc_abstract]
**版本**: V{version}
**数据库方言**: [mysql | postgresql]
**生成日期**: YYYY-MM-DD

---

## 执行顺序

| 序号 | 脚本文件 | 描述 | 预估耗时 | 是否需要停机 |
|------|---------|------|---------|-------------|
| 1 | V{version}__alter_tables.sql | 表结构变更 | [X]min | [是/否] |
| 2 | V{version}__create_tables.sql | 新表创建 | [X]min | 否 |
| 3 | V{version}__init_data.sql | 初始数据/迁移 | [X]min | 否 |

## 前置条件

- [ ] 数据库备份已完成
- [ ] 回滚脚本已验证
- [ ] 执行窗口已确认

## 回滚方案

按执行顺序 **逆序** 执行各脚本中的 ROLLBACK 注释：

1. 执行 `V{version}__init_data.sql` 中的 ROLLBACK 语句
2. 执行 `V{version}__create_tables.sql` 中的 ROLLBACK 语句
3. 执行 `V{version}__alter_tables.sql` 中的 ROLLBACK 语句

## 验证步骤

| 步骤 | 验证内容 | 验证方式 |
|------|---------|---------|
| 1 | 表结构正确 | `DESCRIBE [table_name]` |
| 2 | 索引存在 | `SHOW INDEX FROM [table_name]` |
| 3 | 约束生效 | `SHOW CREATE TABLE [table_name]` |
| 4 | 数据完整 | `SELECT COUNT(*) FROM [table_name]` |
```
