# CPU-submit：RISC-V CPU 课程提交仓库

本仓库是 RV32IM CPU 课程项目的**提交物**：助教下发的框架本体 + 我们生成的 Verilog。
它 fork 自 `ACMClassCourse-2025/RISC-V-CPU-2026`，因此保留了与官方仓库的关联，
框架或测试用例更新时可以直接同步：

```sh
git pull upstream main
```

> 原先的 `README.md` 是指向 `README-EN.md` 的符号链接，本文件取代了它。
> 官方的中文/英文说明仍然保留在 `README-ZH.md` 与 `README-EN.md`，内容未作改动。

## 源码在哪里

CPU 用 **Chisel** 编写，源码不在本仓库，而在开发仓库 `Alastor64/CPU0`：

| 位置 | 内容 |
| --- | --- |
| `CPU0/v0/` | CPU 本体的 Chisel 源码，顶层是 `cpu.StudentTop` |
| `CPU0/design/` | 架构设计说明（微架构、参数、取舍） |
| `CPU0/learn/` | 学习 Chisel 用的练习工程 |
| `CPU0/scripts/` | 生成 / 仿真 / 综合 / 发布脚本 |

本仓库的 `verilog/generated/student_top.sv` 是**生成物，不要手工修改**——
每次重新生成都会覆盖它。

## 怎么生成 RTL

在开发仓库里执行：

```sh
scripts/publish.sh            # 重新生成 student_top.sv 并复制到本仓库，维护 filelist
scripts/publish.sh --commit   # 额外在本仓库把这两个文件提交一次
```

提交信息形如 `Generate Verilog from CPU0@<hash>`，其中 `<hash>` 就是本次 RTL
对应的 Chisel 源码版本，便于复盘以及 Code Review 时追溯。

## 怎么本地编译与测试

硬件工具链（Verilator / Yosys / OpenSTA / ASAP7 库）不在本仓库里，需要自己准备。
有三种情形，按你的情况选一种：

**1. 只有本仓库（例如新机器、新队友）：把 AppImage 放到本仓库根目录。**

框架的 `config.mk` 默认就在这个位置找它，放好之后所有 make 目标开箱即用。
该文件已被框架的 `.gitignore` 忽略，不会进入提交。

```sh
cp /path/to/cpu2026-tools-x86_64.AppImage .   # 放在本仓库根目录
make build
```

**2. 已经克隆了开发仓库 CPU0：复用它解包好的工具链（我们日常用这种方式）。**

好处是不用每次挂载 AppImage、也不往系统里装任何东西。注意解包产物在开发仓库的
`tools/` 下且被 git 忽略，新机器要先跑一次 `scripts/setup-tools.sh`（该脚本本身
需要一个 AppImage 文件）。

```sh
source ../CPU0/scripts/env.sh   # 本地开发仓库目录名可能是 my，按实际路径调整
make build
```

**3. OJ 评测时什么都不用做**：评测环境自带 Verilator，本地工具链只用于自测。

常用命令（上面两种方式都适用）：

```sh
make build                      # 编译周期精确仿真器，产物 build/sim
make test                       # 跑全部正确性用例
make test Case=correctness_add_to_100
make perf                       # 跑性能基准并输出 IPC 几何平均
make synth                      # Yosys + ASAP7 面积，OpenSTA 时序与最高频率
```

## 当前状态

`verilog/generated/student_top.sv` 目前是**接口骨架**：模块名与 AXI4-Lite 端口
完全符合课程要求，但所有 `valid`/`ready` 都拉低，不发起任何访存，因此正确性
用例会以 `timeout` 失败。它的作用是先把
「Chisel 生成 SystemVerilog → 框架编译 → 测试脚本运行」这条链路证明通，
真正的乱序执行核心会在架构确定后逐步替换它。

## 设计文档与报告

架构探索、参数敏感度分析、以及开发过程中的取舍记录，都在开发仓库 `CPU0`
的 `design/` 与报告文档中；本仓库只保留评测所需的框架与 RTL。
