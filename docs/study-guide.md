# rCore Rust OS Lab 学习指南（MyrCoreLab）

> 本文件面向「用 Rust 从零写一个运行在 RISC-V 上的类 Unix 内核」这一实验（即 rCore-Tutorial-v3，版本 3.6.0）。
> 内容均核对自本仓库两个 submodule 的真实结构：`lab/`（实验代码）与 `lecture/`（教程书源码）。

---

## 0. 本仓库结构 & 怎么用

`edidada/MyrCoreLab` 是你的个人学习总仓（umbrella），用 git submodule 组织两个你 fork 的仓库：

| 路径 | 含义 | 指向 |
| --- | --- | --- |
| `lab/` | 实验代码（你写内核的地方） | fork：`edidada/rCore-Tutorial-v3`（上游 `rcore-os/rCore-Tutorial-v3`） |
| `lecture/` | 教程书/讲义源码（reStructuredText） | fork：`edidada/rCore-Tutorial-Book-v3`（上游 `rcore-os/rCore-Tutorial-Book-v3`） |
| `docs/study-guide.md` | 本学习指南 | 本地文档 |

常用操作：

```bash
# 进入实验目录，在 QEMU 启动内核（需先配好环境）
cd lab/os && make run
# 退出 QEMU：Ctrl+A 然后按 X

# 想看某章完整参考实现（lab 仓库有 ch1..ch9 分支，每个分支是到该章为止的完整代码）
cd lab && git checkout chX        # X = 1..9
# 切回你自己的工作分支
cd lab && git checkout lab

# 把子模块拉到上游最新
git submodule update --remote
```

注意：你 fork 的 `lab` 仓库已建好 `lab` 分支并 push 到 GitHub，后续实验代码都提交到这个分支；`lab` 分支目前指向与 `main` 相同的初始提交。

---

## 1. 这门课主要知识点

rCore-Tutorial-v3 的章节与每一章最终形成的「操作系统形态」一一对应（名称取自 `lab/README.md` 的 OS-API-DOCS）：

- **第零章 绪论**：项目背景、操作系统概念总览、环境搭建（Rust / QEMU / RISC-V 工具链）。
- **第一章 应用程序与基本执行环境（`Lib-OS`，库操作系统）**：裸机程序、`#![no_std]` 移除标准库、链接脚本与内存布局（`linker-qemu.ld`）、内核入口第一条指令、基于 RustSBI 的 `console` 输出与 `shutdown`、函数调用约定。
- **第二章 批处理系统（`Batch-OS`）**：特权级（S/U 模式）、Trap 机制、中断与异常处理、上下文切换、系统调用（syscall）接口、批处理加载多个用户程序。
- **第三章 多道程序与分时多任务（`MultiProg-OS` / `TimeSharing-OS`）**：多道程序装载、任务（Task）抽象、协作式 vs 抢占式、时间片轮转（Round-Robin）调度、任务上下文保存/恢复。
- **第四章 地址空间（`AddrSpace-OS`）**：物理/虚拟内存、SV39 分页、页表建立与映射、内核/用户地址空间隔离。
- **第五章 进程（`Process-OS`）**：进程控制块（TCB/PCB）、进程创建与执行（spawn / fork-exec 思路）、进程调度与切换、进程间关系。
- **第六章 文件系统（`FileSystem-OS`）**：块设备与块缓存、easy-fs 简易文件系统、inode / 目录 / 文件描述符、标准 I/O 重定向、在 QEMU 中挂载文件系统镜像。
- **第七章 进程间通信（`IPC-OS`）**：管道（pipe）、信号机制（signal，当前仅支持单线程 app）等。
- **第八章 同步互斥（`SyncMutex-OS`，单处理器）**：临界区、互斥锁（Mutex）、自旋锁、`UPSafeCell` 取代 `RefCell`/`spin::Mutex` 管理静态可变状态、死锁、信号量（Semaphore）、条件变量/管程，以及 race 相关测试。
- **第九章 I/O 设备驱动（`IODevice-OS`）**：基于中断的设备驱动、UART 串口、块设备、键盘、鼠标、GPU 等外设驱动。

贯穿全书的核心能力：Rust `no_std` 裸机开发、所有权/借用与裸机内存安全、内联汇编（`asm!`）、`UPSafeCell` 模式、SBI（RustSBI）接口的使用。

> 教程书（`lecture/source/`）同样是 `chapter0`–`chapter9` 加附录（`final-lab`、`appendix-a`~`appendix-f`、`terminology`）。Part2「Do it better!」目前为空，留作进阶扩展。

---

## 2. 学习路线

1. **环境搭建**（第零章章末 `5setup-devel-env`）：装 Rust、RISC-V 目标、QEMU 7.0、RISC-V GNU 工具链（含 GDB）。
2. **顺序阅读** `lecture/source/chapter0` → `chapter9`，每章末尾有练习（`7exercise`）与答案（`8answer`）。
3. **同步动手（在 `lab/` 上）**：
   - 先 `cd lab && git checkout ch1` 读最小内核，理解后再 `git checkout lab` 自己实现；
   - 逐章推进 `ch2`→`ch9`，每章先把参考分支读懂，再在本仓库 `lab` 分支落地自己的实现。
4. **每章验证**：`cd lab/os && make run` 在 QEMU 跑起来，通过串口输出观察，必要时用 GDB 调试。
5. **（可选）真实硬件 Kendryte K210**：切到 `k210` 分支，`make run BOARD=k210`；第 6 章后需 `make sdcard` 烧写文件系统镜像。
6. **完成综合大实验** `lecture/source/final-lab.rst`（final-lab）。

命令速查：

```bash
cd lab/os && make run        # 构建并运行内核（QEMU）
cd lab && git checkout chX    # 看第 X 章参考实现（X=1..9）
cd lab && git checkout lab    # 回到自己的工作分支
git submodule update --remote # 把 lab/、lecture/ 拉到上游最新
```

---

## 3. 先修课程 / 前置知识

**Rust 语言基础（强烈建议先修，事半功倍）**：

- 清华暑期《Rust 程序设计》课程：<https://lab.cs.tsinghua.edu.cn/rust/>（讲义涵盖语法、所有权、泛型、并发）。
- Rustlings 练习（约 110 道，训练营第一阶段）：<https://github.com/rust-lang/rustlings>。
- 官方 Rust Book：<https://doc.rust-lang.org/book/>。

**实验必需的工具链**（无背景也可边做边学，项目官方定位即「面向无体系结构/汇编/OS 背景的初学者」）：

- Rust + `rustup target add riscv64gc-unknown-none-elf`
- `cargo install cargo-binutils` + `rustup component add llvm-tools-preview` + `rustup component add rust-src`
- QEMU 7.0.0（`riscv64-softmmu` 与 `riscv64-linux-user`）
- RISC-V GNU Embedded Toolchain（含 `riscv64-unknown-elf-gdb`）

**有帮助但非必需**：计算机组成 / RISC-V 汇编、操作系统概论（进程/内存/文件系统概念）、C 语言基础。

---

## 4. 后续精进路线

- 完成 Part2「Do it better!」与 `final-lab` 综合实验。
- **进阶主干项目**（rcore-os 社区，更复杂、功能更全、用到更多新技术）：
  - **zCore**：用 Rust 异步原语重写 Zircon 内核（超时取消等机制）。
  - **arceos**：探索 OS 模块化/组件化架构。
  - **大 rCore**（`rcore-os/rCore`）：Rust 版 THU uCore，Linux 兼容内核。
- **参加开源操作系统训练营**：2026 春夏季 <https://opencamp.cn/os2edu/camp/2026spring> 、2026 秋冬季 <https://opencamp.cn/os2edu/camp/2026fall> ；往届参考仓库 `LearningOS/rust-based-os-comp2025`。
- **深入方向**：SMP 多核、异步 I/O、设备树（DTB/fdt）、形式化验证、把内核移植到更多真实板子。
- **贡献**：给 `rCore-Tutorial-v3` / `rCore-Tutorial-Book-v3` 提 Issue/PR（交流 QQ 群 **735045051**）。

---

## 5. 参考资料

**仓库（本仓库 submodule 指向你的 fork）**：

- 实验代码 fork：`edidada/rCore-Tutorial-v3`（上游 `rcore-os/rCore-Tutorial-v3`）
- 教程书 fork：`edidada/rCore-Tutorial-Book-v3`（上游 `rcore-os/rCore-Tutorial-Book-v3`）

**在线文档**：

- 教程书（网页版）：<https://rcore-os.github.io/rCore-Tutorial-Book-v3/>
- 各章 OS API 文档（rustdoc，取自 `lab/README.md`）：
  - ch1 Lib-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch1/os/index.html>
  - ch2 Batch-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch2/os/index.html>
  - ch3 MultiProg-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch3-coop/os/index.html>
  - ch3 TimeSharing-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch3/os/index.html>
  - ch4 AddrSpace-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch4/os/index.html>
  - ch5 Process-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch5/os/index.html>
  - ch6 FileSystem-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch6/os/index.html>
  - ch7 IPC-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch7/os/index.html>
  - ch8 SyncMutex-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch8/os/index.html>
  - ch9 IODevice-OS：<https://rcore-os.github.io/rCore-Tutorial-v3/ch9/os/index.html>
- Rust 系统编程学习资源（rcore-os wiki）：<https://github.com/rcore-os/rCore/wiki/study-resource-of-system-programming-in-RUST>
- 清华 Rust 课程：<https://lab.cs.tsinghua.edu.cn/rust/>
- 训练营：<https://opencamp.cn/os2edu/camp/2026spring> 、<https://opencamp.cn/os2edu/camp/2026fall>
- 往届参考：<https://github.com/LearningOS/rust-based-os-comp2025>
- 交流 QQ 群：**735045051**

**工具**：Rust toolchain、`cargo-binutils`、QEMU 7.0、RISC-V GNU Toolchain（含 GDB）、Git、GitHub（本仓库用 submodule 组织）。
