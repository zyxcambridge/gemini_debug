# 产品需求文档 (PRD): 神经符号调试代理 (NSDA)

| 文档版本 | 1.0 |
| :--- | :--- |
| **状态** | 初始草案 |
| **最后更新** | 2025-11-22 |

## 1. 产品综述 (Executive Summary)

### 1.1 背景与痛点
现有的 AI 编程助手（如 GitHub Copilot, Cursor, Claude Code）主要依赖文本概率预测代码逻辑。在处理复杂逻辑错误时，由于**缺乏对运行时状态（Runtime State）的直接观测权**，往往产生“幻觉”或无效修复，导致开发者需反复尝试，效率低下。

### 1.2 产品愿景
**神经符号调试代理 (Neuro-Symbolic Debugging Agent, NSDA)** 旨在实现从“基于文本概率的预测”到“基于运行时状态的观测与自治修复”的范式转变。通过深度融合大语言模型（LLM）与符号系统（DAP/形式化验证），构建一个基于证据（Evidence-Based）的下一代智能调试系统。

## 2. 用户角色与场景 (User Personas & Stories)

| 角色 | 典型场景 | 核心需求 |
| :--- | :--- | :--- |
| **资深后端工程师** | 调试复杂的并发竞态条件或深层逻辑错误。 | 需要 AI 能像人类一样下断点、看堆栈，而不是只给建议；需要确保修复不会引入回归 Bug。 |
| **系统级开发者 (x86/C++)** | 面对段错误 (Segfault) 或内存泄漏。 | 需要工具能理解底层内存状态，结合 GDB/LLDB 协议进行精准诊断。 |
| **初级开发者** | 面对报错不知所措，无法复现问题。 | 需要 AI 自动构建最小复现用例 (Reproduction Case) 并解释根本原因。 |

## 3. 功能需求 (Functional Requirements)

### 3.1 核心架构：三元代理协作模型 (Tri-Agent Collaboration)

系统需实现三个专业代理的自主协作：

#### 3.1.1 架构师 (The Architect) - [P0]
*   **职责**：任务编排、策略制定、用户沟通。
*   **功能**：
    *   接收用户自然语言描述的 Bug。
    *   分解调试步骤，指挥 Inspector 和 Verifier。
    *   综合多方信息，生成最终修复方案。

#### 3.1.2 探针 (The Inspector) - [P0]
*   **职责**：运行时观测与诊断（Eyes & Hands）。
*   **功能**：
    *   **DAP 控制器**：通过 MCP 协议操作调试适配器（Debug Adapter）。
    *   **动作集**：设置断点 (Set Breakpoints)、单步执行 (Step Over/Into)、读取变量 (Read Variable)、评估表达式 (Eval)。
    *   **异常捕获**：自动捕获程序崩溃时的堆栈信息。

#### 3.1.3 验证者 (The Verifier) - [P0]
*   **职责**：质量保证与测试驱动（QA）。
*   **功能**：
    *   **测试生成**：编写最小可复现单元测试 (Reproduction Script)。
    *   **静态分析**：集成 CodeQL/ESLint 检查语法和潜在风险。
    *   **形式化验证**：(P1) 利用 SMT 求解器验证补丁逻辑的数学正确性。

### 3.2 核心算法引擎 (Algorithm Engine)

#### 3.2.1 语言代理树搜索 (LATS) - [P0]
*   实现基于蒙特卡洛树搜索 (MCTS) 的推理路径探索。
*   允许代理在发现死胡同（修复失败）时回溯，尝试另一条假设分支。

#### 3.2.2 自我反思 (Reflexion) - [P0]
*   在每次测试失败后，强制 Agent 生成“错误原因分析”，并将其作为短期记忆存入上下文，避免重复犯错。

### 3.3 用户体验与可视化 (UX/UI - Glass Box Design)

#### 3.3.1 时间轴调试视图 (Timeline Debug View) - [P1]
*   将调试会话“视频化”。
*   用户可拖动进度条回溯执行流。
*   **关键事件标注**：AI 自动在时间轴上标记“变量 X 变为 null”、“抛出异常”等关键时刻。

#### 3.3.2 交互式推理图 (Interactive Reasoning Graph) - [P1]
*   可视化 LATS 搜索树。
*   **节点状态**：思考中 (Thinking)、尝试修复 (Patching)、验证通过 (Verified)、失败 (Failed)。
*   **用户干预**：
    *   **剪枝 (Prune)**：用户点击节点标记为“方向错误”，AI 立即停止该分支探索。
    *   **重定向 (Redirect)**：用户向特定节点注入提示（Hint），引导 AI 思考方向。

#### 3.3.3 混合控制模式 (Hybrid Control) - [P0]
*   **自动驾驶 (Autopilot)**：AI 全权负责，直到解决或超时。
*   **总督模式 (Governor Pattern)**：高风险操作（如修改代码、删除文件、执行 Shell 命令）必须经用户点击“批准”。
*   **接管模式 (Takeover)**：用户随时按快捷键暂停 AI，进入标准 IDE 调试模式手动操作。

## 4. 数据与隐私 (Data & Privacy)

### 4.1 本地优先架构 (Local-First)
*   **MCP Server**：运行在用户本地环境（Localhost）。
*   **文件操作**：文件读取、写入、DAP 交互全部在本地完成，不上传完整代码库。

### 4.2 脱敏传输
*   **白名单机制**：仅发送与 Bug 相关的函数片段和堆栈信息给 LLM。
*   **过滤器**：自动识别并掩盖 API Key、Password、ENV 变量等敏感数据。

## 5. 成功指标 (Success Metrics)

| 指标 | 目标值 | 备注 |
| :--- | :--- | :--- |
| **DebugBench 解决率** | > 50% | 超过 GPT-4 Zero-shot 基准 |
| **SWE-bench Lite** | 12% - 15% | 达到 SOTA 水平 |
| **幻觉率 (Hallucination Rate)** | < 5% | 针对变量值的错误引用显著降低 |
| **MTTR (平均修复时间)** | 降低 30% | 对比纯人工调试 |

## 6. 技术栈要求 (Technical Requirements)

*   **IDE 兼容性**：VS Code 插件架构 / Cursor 扩展。
*   **DAP 支持**：优先支持 Python (debugpy), Node.js (js-debug), C/C++ (GDB/LLDB via DAP)。
*   **模型支持**：
    *   Claude 3.5 Sonnet / GPT-4o (推理主力)。
    *   DeepSeek-V3/R1 (高性价比推理选项)。
*   **架构支持**：兼容 x86_64 及 ARM64 架构的调试后端。

