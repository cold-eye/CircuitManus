# CircuitManus - 智能电路设计与交互平台

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Frontend Tech](https://img.shields.io/badge/frontend-FastAPI%20%7C%20WebSocket%20%7C%20JS-brightgreen.svg)]()
[![Status](https://img.shields.io/badge/status-持续迭代中-green.svg)]()

**CircuitManus** (内部代号 OpenManus Agent) 是一个专为电路设计任务构建的、基于 Python 的高级异步智能体，现已集成 FastAPI WebSocket 服务器和现代化 Web UI，提供一个完整的智能交互平台。它利用大语言模型（LLM，当前集成智谱 AI GLM 系列）的强大理解和规划能力，结合一系列内部精确执行的工具，来自动化和辅助完成电路设计相关的操作。其核心架构严格遵循经典的 **感知 -> 规划 -> 行动 -> 观察 -> 响应生成** 的智能体循环模型，并具备强大的容错和自我修正能力。

**平台通过 WebSocket 实现后端 Agent 核心与前端 Web 界面的无缝实时交互。**

---

## ✨ 核心特性

*   **🌐 现代化 Web 用户界面:**
    *   **实时交互:** 基于 FastAPI 和 WebSocket，实现与 Agent 核心的流畅、实时双向通信。
    *   **精致设计:** 美观、清晰、响应式的用户界面，支持浅色/深色/自动主题切换。
    *   **会话管理:** 支持创建、切换、命名和删除多个独立的聊天会话，所有会话数据持久化存储在浏览器 `localStorage`。
    *   **动态状态展示:** 实时显示 Agent 的处理阶段、思考过程摘要、工具执行详情等。
    *   **统一的思考与回复:** Agent 的思考过程（可配置显示）与其最终回复一同展示在聊天气泡中，增强透明度。
    *   **增强的消息区分:** 通过头像和样式清晰区分用户与 Agent 消息。
    *   **文件上传预览:** 支持（概念上的）文件附件，并在发送前进行预览。
    *   **可配置体验:** 提供设置模态框，允许用户调整主题、字号、动画级别、自动滚动等。
    *   **详细处理日志:** UI 内嵌可折叠的处理日志区域，完整记录 Agent 状态更新。

*   **🧠 智能规划与重规划 (Agent 核心):**
    *   **LLM驱动的规划:** 利用 LLM 理解复杂指令，生成包含工具调用或直接回复的结构化 JSON 计划。
    *   **失败后自动重规划:** 当工具执行失败时，Agent 将失败信息反馈给 LLM，请求生成修正后的新计划，极大提升任务成功率。
    *   **LLM规划调用重试:** 在与 LLM 沟通不畅时自动重试。

*   **🛠️ 精确工具执行与容错 (Agent 核心):**
    *   **动态工具注册:** 通过简单的 `@register_tool` 装饰器即可为 Agent 添加新功能。
    *   **异步工具执行:** 工具按计划顺序异步执行，支持工具级重试。
    *   **执行失败中止:** 单个工具在重试后仍失败时，会中止当前计划的后续步骤。

*   **💡 增强的 Agent 逻辑 (Agent 核心):**
    *   **强化直接问答处理:** 即使对于概念性问题或普通对话，Agent 也能正确理解并（通过规划）直接生成回复。
    *   **回调驱动的状态更新:** Agent 核心通过回调函数将所有处理阶段的状态异步发送给服务器，取代了直接的控制台输出，完美适配 WebSocket。
    *   **思考内容分离:** Agent 的思考过程（`<think>`标签内容）从最终回复中逻辑分离，通过专门的状态回调消息发送，便于前端灵活展示。

*   **💾 状态与记忆管理 (Agent 核心):**
    *   **对象化电路状态 (`Circuit` 类):** 清晰封装电路元件、连接和ID生成逻辑。
    *   **多层记忆系统 (`MemoryManager` 类):** 短期记忆（对话历史）、长期记忆（关键操作记录）、统一管理 `Circuit` 对象。

*   **🔧 工程实践 (Agent 核心 & 服务器):**
    *   **异步核心 (`asyncio`):** FastAPI 与 Agent 核心均充分利用异步特性。
    *   **模块化设计:** Agent 核心代码结构清晰，分为 Orchestrator, MemoryManager, LLMInterface, OutputParser, ToolExecutor 等组件。服务器 `server.py` 负责 WebSocket 连接管理和 Agent 实例化。
    *   **详细文件日志:** Agent 每次运行自动在 `agent_logs` 目录下生成带时间戳和 PID 的日志文件，完整记录 Agent 的思考、决策、工具执行详情及错误信息。服务器日志由 Uvicorn/FastAPI 处理。

---

## 🚀 快速开始

### 环境要求

*   Python 3.8 或更高版本
*   Node.js 和 npm/yarn (仅当您需要修改前端 `script.js` 并希望使用构建工具时，本项目当前直接使用原生 JS)
*   现代 Web 浏览器 (Chrome, Firefox, Edge, Safari 最新版)
*   智谱 AI API 密钥 (用于连接大语言模型)

### 安装与运行步骤

1.  **克隆仓库：**
    ```bash
    git clone https://github.com/singularguy/CircuitManus.git # 请替换为您的实际仓库地址
    cd CircuitManus
    ```

2.  **后端 (Python Agent) 设置：**
    *   **安装依赖：**
        核心依赖包括 `zhipuai` (for LLM), `fastapi` (for web server), `uvicorn` (for running FastAPI), `websockets` (FastAPI uses this).
        ```bash
        pip install "zhipuai>=2.0" fastapi uvicorn websockets
        ```
        *建议创建一个 `requirements.txt` 文件并使用 `pip install -r requirements.txt`。*

    *   **配置 API 密钥：**
        您需要提供智谱 AI 的 API Key。
        *   **推荐方式 (环境变量):** 在您的操作系统中设置环境变量 `ZHIPUAI_API_KEY`。
            *   Linux/macOS: `export ZHIPUAI_API_KEY="你的API密钥"`
            *   Windows (CMD): `set ZHIPUAI_API_KEY=你的API密钥`
            *   Windows (PowerShell): `$env:ZHIPUAI_API_KEY="你的API密钥"`
        *   如果未设置，Agent 将无法连接 LLM，功能会受限。

3.  **启动服务器：**
    进入项目根目录，运行 FastAPI 服务器。
    ```bash
    uvicorn server:app --host 127.0.0.1 --port 8000 --reload
    ```
    *   `--reload` 选项用于开发时自动重载代码更改。生产环境请移除。
    *   服务器启动后，您将在控制台看到 Uvicorn 和 FastAPI 的日志，包括 Agent 核心模块的初始化信息。

4.  **访问 Web UI：**
    打开您的 Web 浏览器，访问 `http://127.0.0.1:8000`。
    您应该能看到 CircuitManus 的 Web 界面。应用会自动尝试连接到 WebSocket 服务器。

### Web UI 交互示例

*   **初始化:** 页面加载后，会自动连接 WebSocket。成功后会收到 "通讯链路已建立，Agent已准备就绪!" 的 Toast 提示。
*   **发送消息:** 在底部的输入框输入您的指令 (例如："给我加个1k电阻R1和3V电池B1")，然后点击发送按钮或按 Enter。
*   **查看状态与回复:**
    *   **处理日志区域 (可折叠):** 会实时显示 Agent 的规划、工具调用、思考等详细步骤。
        *   例如："开始规划任务..." -> "LLM思考过程 (规划阶段): 用户要求添加两个元件..." -> "操作 'Add Component' 开始..." -> "操作 'Add Component' 完成。" -> "正在生成最终回复..." -> "最终回复生成完成。"
    *   **聊天区域:**
        *   您的消息会显示在右侧。
        *   Agent 的回复会显示在左侧，如果开启了“在日志中显示思考过程” (设置项，默认为开启)，Agent 回复气泡的顶部会先展示其最终的思考过程，然后是正式的回复文本。
        *   例如，对于 "添加电阻R1和电池B1" 的指令，Agent 回复可能如下：
            ```
            [Agent Avatar]
            [思考过程] 用户要求添加两个元件：R1和B1。规划将调用add_component_tool两次。工具都成功了。现在总结回复。
            --------------------
            好的，我已经为您添加了电阻 R1 (值: 1k) 和电池 B1 (值: 3V)。
            ```
*   **会话管理:**
    *   点击侧边栏的 "新建对话" 创建新会话。
    *   在 "会话历史" 中点击不同会话进行切换。
    *   点击聊天顶部的会话名称旁的编辑按钮可重命名当前会话。
*   **设置:** 点击侧边栏的齿轮图标打开设置，可以调整主题、字号、动画、思考过程显示等。

---

## 🛠️ Agent 可用工具 (内部 Actions)

Agent 通过调用内部注册的工具来执行具体操作。当前包含以下核心工具：

*   **`add_component_tool`**: 添加一个新的电路元件（如电阻、电容、电池等）。支持自动或手动指定 ID 和可选的值。
*   **`connect_components_tool`**: 使用元件 ID 连接两个已存在的元件。
*   **`describe_circuit_tool`**: 获取当前电路中所有元件和连接的详细文本描述。
*   **`clear_circuit_tool`**: 彻底清空当前电路的所有内容。

*开发者可以通过在 `CircuitManusCore.py` 中使用 `@register_tool` 装饰器轻松添加更多自定义工具。*

---

## 📁 项目结构 

```
CircuitManus/
├── WebUIAgentlogs/         # Agent 核心运行日志 (自动生成)
├── static/                 # 存放前端静态文件
│   ├── style.css           # UI 样式表
│   └── script.js           # UI 交互逻辑
|   └── index.html          # Web UI 入口页面
├── CircuitManusCore.py     # Agent 核心逻辑
├── server.py               # FastAPI WebSocket 服务器
└── README.md               # 本文件
```

---

## 💡 技术栈

*   **后端 (Agent & Server):**
    *   Python 3.8+
    *   FastAPI: 高性能 Web 框架，用于构建 API 和 WebSocket 服务器。
    *   Uvicorn: ASGI 服务器，用于运行 FastAPI 应用。
    *   ZhipuAI SDK: 用于与智谱 GLM 大语言模型交互。
    *   Asyncio: Python 标准库，用于异步编程。
*   **前端 (Web UI):**
    *   HTML5
    *   CSS3 (原生，未使用框架)
    *   JavaScript (原生 ES6+，未使用框架)
    *   Font Awesome: 图标库。
    *   Animate.css: CSS 动画库。
*   **核心概念:**
    *   Agentic Loop (Perceive-Plan-Act-Observe-Respond)
    *   LLM-based Planning
    *   Tool Use / Function Calling (simulated via structured JSON)
    *   WebSocket Real-time Communication

---

## 📜 开源协议

本项目基于 **MIT 许可证** 开源。这意味着您可以自由地使用、复制、修改、合并、出版、分发、再许可和/或销售本软件的副本，只需在所有副本或重要部分中包含原始的版权声明和许可声明。

详情请参阅仓库根目录下的 `LICENSE` 文件（如果未来添加）或访问 [MIT License](https://opensource.org/licenses/MIT) 官方说明。

---

## 🤝 贡献与反馈

我们热烈欢迎各种形式的贡献，包括但不限于：

*   **报告 Bug 或提出功能建议:** 请通过您项目仓库的 GitHub Issues 功能。
*   **提交代码改进:** 请遵循标准的 Pull Request 流程。
*   **完善文档与教程:** 帮助新用户更快上手。
*   **分享使用案例:** 展示 CircuitManus 如何帮助您完成工作。

如果您在使用中遇到任何问题或有任何想法，请不要犹豫，在项目的 GitHub Issues 中让我们知道！

---

*CircuitManus - 驱动智能，点亮电路设计的未来！ Let's Seek The X*
