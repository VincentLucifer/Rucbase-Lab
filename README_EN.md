# Rucbase Lab · Database Systems Coursework

[简体中文](README.md) | [English](README_EN.md)

C++ coursework based on the **Rucbase teaching database framework**, focused on disk pages, buffer pools, LRU / Clock replacement, record management and B+ tree indexes. Modules live directly at the repository root so readers can browse the database kernel from storage through query execution.

**Stack:** C++17 · CMake · GoogleTest · Flex / Bison · Bash

> This repository retains course source code and experiment materials. Some executor, transaction and recovery interfaces remain unfinished. Included tests and reference outputs do not establish that all tests pass.

## Repository structure

```text
Rucbase-Lab/
├── common/           # Configuration, context, logging utilities and read/write latches
├── storage/          # Disk pages and buffer-pool management
├── replacer/         # LRU / Clock page replacement
├── record/           # Record files, slot bitmaps and scans
├── index/            # B+ tree nodes, index operations and scans
├── parser/           # SQL lexer, grammar and abstract syntax tree (AST)
├── system/           # Database, table, index and metadata management
├── execution/        # Executors, SQL inputs, reference outputs and test scripts
├── transaction/      # Transaction and concurrency-control lab interfaces
├── recovery/         # Logging, checkpoint and recovery lab interfaces
├── CMakeLists.txt    # Module configuration for the course parent project
├── rucbase.cpp       # Course server entry-point source
├── interp.h          # SQL interpretation and module coordination
├── defs.h            # Shared types
├── errors.h          # Exception definitions
├── record_printer.h  # Record output formatting
├── README.md
└── README_EN.md
```

## Core implementations

| Module | Implementation and focus | Entry point |
| --- | --- | --- |
| Disk management | Page reads/writes, file operations and page allocation | [disk_manager.cpp](storage/disk_manager.cpp) |
| Buffer pool | Page table, free frames, pin counts, dirty-page writeback and page lifecycles | [buffer_pool_manager.cpp](storage/buffer_pool_manager.cpp) |
| Page replacement | LRU and Clock behind a shared `Replacer` interface | [lru_replacer.cpp](replacer/lru_replacer.cpp) / [clock_replacer.cpp](replacer/clock_replacer.cpp) |
| Record management | Slot bitmaps for page space; `Rid(page_no, slot_no)` for record locations | [rm_file_handle.cpp](record/rm_file_handle.cpp) |
| B+ tree indexes | Lookup, insertion/splitting, deletion/coalescing/redistribution, root adjustment and leaf scans | [ix_node_handle.cpp](index/ix_node_handle.cpp) / [ix_index_handle.cpp](index/ix_index_handle.cpp) |

In the buffer pool, pin counts track active page use and dirty flags track writeback requirements. Record management and indexes use these page interfaces to access persistent data.

## Queries and course extensions

[parser/](parser/) provides the SQL lexer, grammar and AST; [system/](system/) manages table and index metadata; [execution/](execution/) retains scan, projection, join and record-modification executor sources. Some executors still contain unfinished course tasks.

[transaction/](transaction/) and [recovery/](recovery/) retain transaction, locking, logging and checkpoint interfaces. Methods including transaction commit/abort and recovery redo/undo still have empty implementations.

[input_all_task.sql](execution/input_all_task.sql) contains course SQL inputs such as:

```sql
create table student (id int, name char(32), major char(32));
create index student (id);
insert into student values (1, 'Tom', 'Computer Science');
select * from student where id >= 1;
update student set major = 'Electrical Engineering' where id = 1;
```

## Getting the source and building

```bash
git clone https://github.com/VincentLucifer/Rucbase-Lab.git
cd Rucbase-Lab
```

The code uses Linux / POSIX interfaces; the course Linux environment or WSL can provide these. Dependencies include a C++17 compiler, CMake, GoogleTest, Flex / Bison and the system libraries required by the parent project.

**This repository is not a complete standalone build project.** The root `CMakeLists.txt` is the original course source-directory module configuration. It lacks the parent project's `project()`, dependency initialisation and output-directory setup; the parent project must also provide the GoogleTest targets.

When integrating with an existing course project, point its `add_subdirectory(...)` at this repository root in place of the original course `src/` directory, retaining the parent's dependencies and compiler settings. Once configured, build the SQL experiment target from the parent project directory:

```bash
cmake --build build --target exec_sql
```

## Test materials

GoogleTest sources sit alongside their modules and cover course tasks for disks, buffers, LRU / Clock, records and B+ trees. See each module's `CMakeLists.txt` for available targets.

SQL experiment instructions are in [execution/README.md](execution/README.md). Run the Bash scripts from this repository's `execution/` directory:

```bash
cd execution
# Defaults to this repository's build/bin/exec_sql; override with the parent's absolute output path.
EXEC_SQL_BIN=/absolute/path/to/course/build/bin/exec_sql bash taskall_test.sh
```

The scripts execute SQL statements and use `diff` to compare results with reference files. They create/remove `ExecutorTest_db` and overwrite `output.txt` in the working directory; run them from the directory shown above.

## Suggested reading order

1. [storage/page.h](storage/page.h) → [storage/disk_manager.cpp](storage/disk_manager.cpp): page layout and disk access.
2. [storage/buffer_pool_manager.cpp](storage/buffer_pool_manager.cpp) → [replacer/](replacer/): pinning, eviction and dirty-page writeback.
3. [record/rm_file_handle.cpp](record/rm_file_handle.cpp): records organised into pages and slots.
4. [index/ix_node_handle.cpp](index/ix_node_handle.cpp) → [index/ix_index_handle.cpp](index/ix_index_handle.cpp): node operations and tree maintenance.
5. [interp.h](interp.h) → [parser/](parser/) / [execution/](execution/): how SQL connects to storage and index interfaces.

## Background

Database coursework based on the Rucbase teaching framework, exploring kernel data structures, page-resource lifecycles and module interaction while retaining the course framework, test materials and extension interfaces.
