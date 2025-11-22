# gemini_debug

gemini debug ai code

## Project NSDA（Neuro-Symbolic Debugging Agent）

Project NSDA 定义了下一代智能调试系统。与当前基于文本预测的 AI 工具不同，NSDA 通过调试适配器协议（DAP）直接观测程序运行时状态，让调试从“盲猜”转向“实证”。

- **三元代理架构**：架构师、探针、验证者协作，结合语言代理树搜索（LATS），可自主设置断点、追踪内存变化并形式化验证修复方案。
- **Glass Box UX**：通过“白盒化”交互引入时间旅行调试与动态推理图，支持开发者回放数据流并干预 AI 决策。
- **高可信调试体验**：以运行时证据消除模型幻觉，提供可解释、自动化且高准确率的调试流程。
