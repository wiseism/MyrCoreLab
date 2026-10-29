# 

清华的 Rust OS 相关资源主要分为两类：**清华计算机系开设的 Rust 语言课程**，以及由清华老师发起的**开源操作系统训练营（rCore 实验）**。两者的网址和实验资料如下。

### 清华 Rust 程序设计课程（语言基础）
这是清华计算机系的一门暑期课程，适合在进入 OS 实验前系统学习 Rust 语法。
- **课程网址**：`https://lab.cs.tsinghua.edu.cn/rust/` 
- **相关资料**：网站提供课程讲义（如 2022 年的幻灯片 PDF），内容涵盖基本语法、所有权、泛型、并发等。该课程也常被开源 OS 训练营列为推荐的前置 Rust 学习资源 。

### 开源操作系统训练营 / rCore 实验（OS Lab）
这是由清华陈渝、向勇老师发起的经典项目，通过 Rust 编写 RISC-V 操作系统。训练营按阶段推进，包含 **Rustlings 练习** 和 **rCore-Tutorial 内核实验**。
- **训练营主页（报名与入口）**：
    - 2026 春夏季：`https://opencamp.cn/os2edu/camp/2026spring` 
    - 2026 秋冬季：`https://opencamp.cn/os2edu/camp/2026fall` 
- **核心实验资料（rCore-Tutorial）**：
    - 官方教程书：`https://rcore-os.github.io/rCore-Tutorial-Book-v3/`。这是实验的核心文档，指导从零开始实现一个类 Unix 内核 。
- **Lab 形式**：训练营的实验通常包含 **Rustlings**（约 110 道 Rust 语法练习题）作为第一阶段，以及基于 rCore-Tutorial 的 **专业阶段实验**（涵盖批处理、虚拟内存、文件系统等）。往届的每日学习记录和实验代码通常托管在 GitHub 的 `LearningOS` 组织下（例如 `LearningOS/rust-based-os-comp2025`），可以作为参考 。

如果你是想找**清华校内操作系统课程（非训练营）的实验**，通常使用的是 ucore 或 rCore 实验框架，相关资料一般通过清华网络学堂或课程主页分发，校外公开访问的稳定入口较少，建议以训练营的 rCore 教程作为主要实践途径。

