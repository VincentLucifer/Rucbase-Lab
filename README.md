# Rucbase Lab · 数据库系统课程实践

[简体中文](README.md) | [English](README_EN.md)

基于 **Rucbase 教学数据库框架**的 C++ 课程实验，重点实现磁盘页面管理、缓冲池、LRU / Clock 置换、记录管理与 B+ 树索引。各模块直接放在仓库根目录，便于浏览数据库内核从存储到查询执行的组织方式。

**技术栈：** C++17 · CMake · GoogleTest · Flex / Bison · Bash

> 本仓库保留课程源码与实验材料。部分执行器、事务与恢复模块仍有待完成的实验接口；仓库中的测试源码与参考输出不代表全部测试已通过。

## 仓库结构

```text
Rucbase-Lab/
├── common/           # 配置、上下文、日志工具与读写锁
├── storage/          # 磁盘页面与缓冲池管理
├── replacer/         # LRU / Clock 页面置换策略
├── record/           # 记录文件、slot 位图与记录扫描
├── index/            # B+ 树节点、索引操作与扫描
├── parser/           # SQL 词法、语法与抽象语法树（AST）
├── system/           # 数据库、表、索引与元数据管理
├── execution/        # 执行器、SQL 输入、参考输出与测试脚本
├── transaction/      # 事务管理与并发控制实验接口
├── recovery/         # 日志、检查点与恢复实验接口
├── CMakeLists.txt    # 供课程父工程集成的模块配置
├── rucbase.cpp       # 课程服务端入口源码
├── interp.h          # SQL 解释与模块调度
├── defs.h            # 公共类型定义
├── errors.h          # 异常定义
├── record_printer.h  # 记录输出格式
├── README.md
└── README_EN.md
```

## 核心实现

| 模块 | 实现与关注点 | 源码入口 |
| --- | --- | --- |
| 磁盘管理 | 页面读写、文件操作与页面分配 | [disk_manager.cpp](storage/disk_manager.cpp) |
| 缓冲池 | 页表、空闲帧、pin count、脏页写回与页面生命周期 | [buffer_pool_manager.cpp](storage/buffer_pool_manager.cpp) |
| 页面置换 | 通过统一 `Replacer` 接口实现 LRU 与 Clock | [lru_replacer.cpp](replacer/lru_replacer.cpp) / [clock_replacer.cpp](replacer/clock_replacer.cpp) |
| 记录管理 | 用 slot 位图管理页内空间，以 `Rid(page_no, slot_no)` 定位记录 | [rm_file_handle.cpp](record/rm_file_handle.cpp) |
| B+ 树索引 | 查找、插入分裂、删除合并 / 重分配、根调整与叶子扫描 | [ix_node_handle.cpp](index/ix_node_handle.cpp) / [ix_index_handle.cpp](index/ix_index_handle.cpp) |

缓冲池中，pin count 表示页面是否仍被使用，dirty 标记表示是否需要写回磁盘；记录管理与索引通过这些页面接口访问持久化数据。

## 查询与课程拓展

[parser/](parser/) 提供 SQL 词法、语法和 AST；[system/](system/) 管理表与索引元数据；[execution/](execution/) 保留扫描、投影、连接及记录修改等执行器源码。部分执行器仍包含待完成的课程任务。

[transaction/](transaction/) 和 [recovery/](recovery/) 保留事务、锁、日志与检查点相关接口，其中事务提交 / 回滚及恢复重做 / 撤销等方法仍有空实现。

[input_all_task.sql](execution/input_all_task.sql) 包含课程 SQL 实验输入，例如：

```sql
create table student (id int, name char(32), major char(32));
create index student (id);
insert into student values (1, 'Tom', 'Computer Science');
select * from student where id >= 1;
update student set major = 'Electrical Engineering' where id = 1;
```

## 获取与构建

```bash
git clone https://github.com/VincentLucifer/Rucbase-Lab.git
cd Rucbase-Lab
```

代码依赖 Linux / POSIX 接口，可使用课程 Linux 环境或 WSL。课程环境需要 C++17 编译器、CMake、GoogleTest、Flex / Bison，以及父工程要求的系统库。

**当前仓库不是完整的独立构建工程。** 根目录的 `CMakeLists.txt` 是原课程源码目录的模块配置，缺少父工程的 `project()`、依赖初始化和输出目录设置；GoogleTest 目标也需要由父工程提供。

集成到已有课程工程时，让父工程的 `add_subdirectory(...)` 指向本仓库根目录，替代原先指向课程源码 `src/` 的配置，并保留父工程的依赖与编译选项。完成配置后，在父工程目录构建 SQL 实验目标：

```bash
cmake --build build --target exec_sql
```

## 测试材料

GoogleTest 源码与对应模块放在一起，覆盖磁盘、缓冲池、LRU / Clock、记录管理和 B+ 树等课程任务。可用目标以各模块的 `CMakeLists.txt` 为准。

SQL 实验说明见 [execution/README.md](execution/README.md)。从本仓库的 `execution/` 目录运行 Bash 脚本：

```bash
cd execution
# 默认使用本仓库 build/bin/exec_sql；父工程输出在其他位置时指定绝对路径。
EXEC_SQL_BIN=/absolute/path/to/course/build/bin/exec_sql bash taskall_test.sh
```

脚本逐条执行 SQL，并使用 `diff` 比较输出与参考文件。运行时会在当前目录创建 / 清理 `ExecutorTest_db`，并重写 `output.txt`；请按上述目录执行。

## 建议阅读顺序

1. [storage/page.h](storage/page.h) → [storage/disk_manager.cpp](storage/disk_manager.cpp)：理解页面布局与磁盘读写。
2. [storage/buffer_pool_manager.cpp](storage/buffer_pool_manager.cpp) → [replacer/](replacer/)：理解页面固定、置换与脏页写回。
3. [record/rm_file_handle.cpp](record/rm_file_handle.cpp)：理解记录在页与 slot 中的组织。
4. [index/ix_node_handle.cpp](index/ix_node_handle.cpp) → [index/ix_index_handle.cpp](index/ix_index_handle.cpp)：理解节点操作与树结构维护。
5. [interp.h](interp.h) → [parser/](parser/) / [execution/](execution/)：追踪 SQL 如何连接存储与索引接口。

## 项目背景

这是基于 Rucbase 教学框架的数据库课程实践，围绕数据库内核的数据结构、页面资源生命周期与模块协作展开，保留课程框架、测试材料和拓展实验接口。
