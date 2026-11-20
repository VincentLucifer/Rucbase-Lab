# SQL 执行实验（Lab 3）

[返回项目说明](../README.md)

本目录使用 Linux Bash 脚本逐条执行 SQL，再用 `diff` 比较实际输出与参考输出。执行器源码仍保留部分待完成的课程任务，参考输出用于结果对照。

## 实验材料

| 脚本 | SQL 输入 | 参考输出 |
| --- | --- | --- |
| `task1_test.sh` | `input_task1.sql` | `res_task1_output.txt` |
| `task2_test.sh` | `input_task2.sql` | `res_task2_output.txt` |
| `task3_test.sh` | `input_task3.sql` | `res_task3_output.txt` |
| `taskall_test.sh` | `input_all_task.sql` | `res_taskall_output.txt` |

`ExecutorTest_db_task2and3/` 是任务 2、3 使用的数据库样本。`output.txt` 是脚本生成的实际输出。

## 构建与运行

本仓库的 CMake 配置需要集成课程父工程，具体要求见[根目录 README](../README.md#获取与构建)。父工程配置完成后，从父工程目录构建：

```bash
cmake --build build --target exec_sql
```

然后进入本仓库的 `execution/` 目录。脚本默认读取 `../build/bin/exec_sql`；如果父工程将程序输出到其他位置，通过 `EXEC_SQL_BIN` 指定实际的绝对路径：

```bash
cd /absolute/path/to/Rucbase-Lab/execution
EXEC_SQL_BIN=/absolute/path/to/course/build/bin/exec_sql bash task1_test.sh
```

如果程序已位于本仓库的 `build/bin/exec_sql`，可以直接运行：

```bash
bash taskall_test.sh
```

脚本输出 `Pass Success!` 或 `Pass Failed!`，不一致时 `diff` 会显示差异。请使用 Bash 执行，避免使用 `sh`，因为脚本使用了 Bash 字符串语法。

运行脚本会清理当前目录的 `ExecutorTest_db` 和 `output.txt`；任务 2、3 会复制样本数据库。请在上述 `execution/` 目录中运行，并保留参考输出及样本目录。
