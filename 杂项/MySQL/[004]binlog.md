**binlog 是 MySQL 层面的日志**（区别于 InnoDB 引擎层的 redo log），它是记录所有对数据库 **执行了更改操作的 SQL 语句或事件** 的日志文件。

主要用于：

- **主从复制**：主库将 binlog 传输给从库，从库解析并执行以保持数据同步。
- **数据恢复**：基于全量备份和 binlog 增量日志，可将数据库恢复到任意时间点。

## binlog 和 redo log 区别

| 对比项           | **Binlog**                       | **Redo Log**                |
| ---------------- | -------------------------------- | --------------------------- |
| 所属层级         | MySQL Server 层                  | InnoDB 存储引擎层           |
| 日志作用         | 主从复制、数据恢复、增量备份     | 崩溃恢复、事务持久性（WAL） |
| 写入时机         | **事务提交时一次性写入**         | **事务执行中就持续写入**    |
| 写入对象         | 记录 SQL 或数据变更事件          | 记录数据页的物理变化        |
| 是否参与恢复     | ✅ 恢复到某时间点（基于 binlog）  | ✅ 宕机恢复（基于 redo log） |
| 是否依赖事务     | 只有事务提交时才写入             | 不管事务是否成功都可以先写  |
| 是否可用于复制   | ✅ 是主从复制的基础               | ❌ redo log 不参与复制       |
| 是否支持引擎无关 | ✅ 所有支持 binlog 的引擎都能写入 | ❌ 仅适用于 InnoDB 引擎      |

> **binlog 记录的是 “你干了什么”，redo log 记录的是 “你怎么干的”。**

- binlog 像一份操作说明书：“某人于几点对哪行数据做了什么修改”。
- redo log 像一份工程记录：“我在某页某偏移处写了这个值”。

## 事务提交分为两阶段

1. 生成 **redo log** → 写入 redo log buffer（内存）。

2. 生成 **binlog** → 写入 binlog cache（内存）。

3. 提交阶段：

   - 先将 binlog cache 写入磁盘（fsync 受 `sync_binlog` 控制）。

   - 再提交 redo log（写入 redo log 并打 COMMIT 标志）。

4. 成功完成后，事务正式提交。

为防止 **redo log 提交成功但 binlog 丢失**，MySQL 使用了“两阶段提交机制”，**只有 binlog 成功刷盘，redo log 才会提交**。这样保证了：

- 如果 binlog 写入失败，redo log也不提交。
- 如果 redo log 写入失败，事务整体失败，binlog 即使写了也不算提交。因为从库重放 binlog 时，**只会处理 redo log 提交成功的事务对应的 binlog**，否则主从会不一致。

## 只会处理 redo log 提交成功的事务对应的 binlog

事务成功之后，redo log 会添加 COMMIT 标记来表明事务成功。

**MySQL 先扫描 redo log（不是 binlog）**，如果发现某个事务只有 prepare，没有 commit，判断为未完成事务。回滚它，**不恢复它**， 忽略其 binlog 内容。

这样就防止了：

- binlog 有记录
- 但实际上事务没提交成功（因为 redo log 没完成）

## 日志格式

| 格式类型    | 内容                          | 优点           | 缺点                                           |
| ----------- | ----------------------------- | -------------- | ---------------------------------------------- |
| `STATEMENT` | 记录原始 SQL 语句             | 空间小，效率高 | 某些语句可能在主从执行结果不一致（如带 now()） |
| `ROW`       | 记录每行变更的数据值          | 保证主从一致性 | 日志体积大                                     |
| `MIXED`     | 自动选择 `STATEMENT` 或 `ROW` | 两者兼顾       | 逻辑复杂                                       |

## binlog 什么时候刷盘

| **值**  | **刷盘机制**                                                 | **一致性与性能**                                             |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`0`** | **由 MySQL 控制刷盘时机**（依赖操作系统的缓存机制） - 事务提交时，binlog 先写入内存中的 binlog cache，再由操作系统异步刷盘。 | **性能最高**，但风险最大： 若 MySQL 或操作系统崩溃，可能丢失 **未刷盘的所有 binlog 数据**。 |
| **1**   | **每次事务提交时，同步将 binlog 刷入磁盘**（调用 fsync 系统调用）。 | **一致性最强**（完全不丢 binlog），但 **性能略低**（每次提交都需磁盘 I/O） - **默认值**，适用于金融等强一致性场景。 |
| **`N`** | **累计 N 个事务提交后，批量将 binlog 刷入磁盘**（N 为正整数，如 100）。 | **性能与一致性的平衡**： 若崩溃，可能丢失最近 N-1 个事务的 binlog，但减少了刷盘次数。 |

