# 柠檬叔个人助手 (Lemonhall Personal Assistant)

这是一个基于 `agno` 框架构建的个性化命令行 AI 助手，并通过 FastAPI 提供 Web API 接口和一个简单的 ChatUI 前端界面。它模拟了一个由多个专业领域 Agent 组成的团队，由一个总控 Agent 根据用户问题进行路由分发，旨在为"柠檬叔"提供定制化的信息查询和任务处理能力。

## 主要功能

*   **多 Agent 协作**: 包含处理个人信息、技术环境、开发工具、项目管理、服务器配置、AI 术语等多个专业 Agent。
*   **智能路由**: 使用 `agno` 的 `Team` 路由模式，根据问题内容自动选择最合适的 Agent 进行处理 (详见下方【Agno Team 路由机制】部分)。
*   **持久化记忆**: 使用 SQLite (`lemonhall_memory.db`) 存储对话历史和用户记忆，实现跨会话的上下文感知。
*   **个性化知识**: Agent 被预置了关于"柠檬叔"的特定信息（如技术栈、常用工具、项目目录等）。
*   **流式输出**: 结果以流式方式打印到控制台，提供实时反馈。
*   **模型封装**: 将火山引擎 DeepSeek 模型的初始化逻辑封装在 `VolcEngineModelProvider` 类中，方便管理和复用。
*   **Web API**: 通过 FastAPI (`src/api_server.py`) 暴露 `/ask` (POST) 接口，接收用户查询并返回 Agent 响应。
*   **Web 前端**: 提供一个基于 ChatUI 的简单聊天界面 (`frontend/index.html`)，通过 API 与后端交互。
*   **前端托管**: FastAPI 服务器同时负责托管前端静态文件。

## 技术栈

*   **后端**: 
    *   Python (3.12+)
    *   agno: AI Agent 和 Team 框架
    *   DeepSeek (via VolcEngine): 底层大语言模型
    *   FastAPI: Web 框架，用于构建 API 和托管前端
    *   Uvicorn: ASGI 服务器，用于运行 FastAPI 应用
    *   SQLite: 用于持久化记忆
*   **前端**: 
    *   HTML
    *   React (via CDN)
    *   ChatUI (via CDN): 聊天界面库
    *   Babel Standalone (via CDN): 用于在浏览器中转译 JSX (仅开发方便，生产环境应预编译)
*   **开发工具**:
    *   uv: Python 包管理和虚拟环境工具
    *   Git: 版本控制

## 安装与配置

1.  **克隆仓库**:
    ```bash
    git clone <your-repository-url>
    cd chatui-agent-assistant 
    ```

2.  **创建虚拟环境**: (推荐使用 `uv`)
    ```bash
    uv venv
    ```
    激活环境:
    *   Windows (PowerShell): `.venv\Scripts\Activate.ps1`
    *   Linux/macOS: `source .venv/bin/activate`

3.  **安装依赖**: (你需要创建一个 `requirements.txt` 或使用 `pyproject.toml` 配合 `uv`)
    假设你有一个 `requirements.txt`，至少应包含以下内容:
    ```plaintext
    agno
    fastapi
    uvicorn[standard] # [standard] 包含性能优化
    openai # agno 可能依赖
    httpx # agno/openai 可能依赖
    python-multipart # FastAPI CORS 中间件可能需要
    aiofiles # FastAPI StaticFiles 可能需要
    ```
    然后执行安装命令:
    ```bash
    uv pip install -r requirements.txt 
    ```
    *你可以使用 `uv pip freeze > requirements.txt` 来生成当前环境的完整依赖列表。*

4.  **设置环境变量**:
    此项目需要设置火山引擎 DeepSeek 的 API Key。你需要将其设置为环境变量 `DEEPSEEK_API_KEY`。
    *   **Windows (PowerShell - 临时)**:
        ```powershell
        $env:DEEPSEEK_API_KEY = "your_actual_api_key" 
        ```
    *   **Windows (PowerShell - 永久用户级)**:
        ```powershell
        [Environment]::SetEnvironmentVariable("DEEPSEEK_API_KEY", "your_actual_api_key", "User")
        ```
        修改后可能需要重启 PowerShell 或 VS Code。
    *   **Linux/macOS**:
        ```bash
        export DEEPSEEK_API_KEY="your_actual_api_key"
        # 将上面这行添加到 ~/.bashrc 或 ~/.zshrc 文件中可实现永久生效
        ```

## 如何运行

项目提供了两种主要的交互方式：命令行控制台和 Web 界面。

### 1. 运行 Web API 服务器和前端界面 (推荐)

在项目根目录下，启动 FastAPI 服务器：

```powershell
# 确保虚拟环境已激活
uvicorn src.api_server:app --host 0.0.0.0 --port 8000 --reload
```
*   `--reload` 参数会在代码变动时自动重启服务器，方便开发。

服务器启动后：
*   在浏览器中访问 `http://localhost:8000/` 可以看到 ChatUI 前端界面。
*   API 接口文档 (Swagger UI) 位于 `http://localhost:8000/docs`。
*   API 状态端点位于 `http://localhost:8000/api-status`。

### 2. 运行命令行控制台 (用于调试或无前端场景)

如果你只想在命令行中与 Agent 团队交互：

```powershell
# 确保虚拟环境已激活
# 方式一：直接运行脚本 (依赖 src/teams_consoles.py 中的 sys.path hack)
& .venv/Scripts/python.exe src/teams_consoles.py 

# 方式二：作为模块运行 (更规范)
python -m src.teams_consoles
```
启动后，程序会提示你输入问题。输入 "exit" 可以退出程序。

## 技术细节说明

### 前端实现 (ChatUI)

*   前端界面位于 `frontend/index.html`。
*   它使用最简单的方式，通过 CDN 引入 React (v17) 和 ChatUI (v3) 库，以及 Babel Standalone 用于实时转译 JSX。
    *   **注意**: 生产环境中应使用构建工具（如 Vite, Create React App）预编译 JSX 和打包代码。
*   核心逻辑在 `<script type="text/babel">` 块中：
    *   使用 `ChatUI.default` 访问主 `<Chat>` 组件，使用 `ChatUI.Bubble`, `ChatUI.useMessages`, `ChatUI.Typing`, `ChatUI.Avatar` 访问其他具名组件和 Hooks。
    *   `useMessages` hook 用于管理聊天消息列表。
    *   通过 `useState` 自行管理 `isTyping` 状态，以控制"正在输入"提示的显示。
    *   `handleSend` 函数负责：
        1.  将用户消息添加到界面。
        2.  设置 `isTyping = true`。
        3.  使用 `fetch` 调用后端的 `/ask` (POST) API，将用户查询放在 JSON 请求体中。
        4.  处理 API 返回的 JSON 响应（包括成功内容和可能的错误），并将助手回复（包含头像和处理 Agent 名称）添加到界面。
        5.  在 `finally` 块中设置 `isTyping = false`。
    *   `renderMessageContent` 函数根据消息类型（`text` 或 `typing`）渲染不同的内容（气泡或 `Typing` 组件）。
    *   为用户和助手消息配置了 SVG 头像（位于 `frontend/assets/avatars/`）。

### 后端 API 与前端托管 (FastAPI)

*   后端服务器代码位于 `src/api_server.py`。
*   **API 端点**: 主要提供 `/ask` (POST) 接口，接收包含 `query` 的 JSON 请求体，调用 `lemonhall_assistant.run(stream=False)` 处理，并返回包含 `handling_agent` 和 `response` 的 JSON。
*   **CORS 配置**: 使用 `fastapi.middleware.cors.CORSMiddleware` 允许来自开发服务器（如 `localhost:3000`, `localhost:5173`）以及服务器自身的请求，确保浏览器安全策略允许前端调用 API。
*   **前端文件托管**: 
    *   定义了一个 GET 路由 `/`，使用 `fastapi.responses.FileResponse` 直接返回 `frontend/index.html` 的内容。
    *   使用 `app.mount("/", StaticFiles(directory=frontend_dir), name="frontend")` 将整个 `frontend` 目录挂载到根路径，使得浏览器可以请求到 `assets/avatars/user.svg` 等静态资源。
    *   **重要**: `StaticFiles` 挂载必须放在特定的 `/` 路由之后，以确保根路径请求优先由 `FileResponse` 处理。
*   **动态路径添加**: 文件开头的 `sys.path` 修改是为了在直接使用 `uvicorn src.api_server:app` 启动时，也能正确解析 `from src.teams_consoles import ...` 这样的绝对导入。

### Agno Team 路由机制 (`mode="route"`)

本项目中的 `lemonhall_assistant` Team 配置了 `mode="route"`。在这种模式下：

1.  **目标**: Team 的 Leader Agent (由 Team 的 `model` 和 `instructions` 定义) 接收用户请求。
2.  **分析与决策**: Leader Agent 分析用户请求的内容，并结合其对各个成员 Agent 的 `role`（角色描述）的理解，决定哪个 Agent 最适合处理这个请求。
3.  **转发**: Leader Agent 使用内部的 `forward_task_to_member` 工具（或类似机制）将任务以及必要的上下文（可能包括原始请求和期望输出的描述）转发给选定的成员 Agent。
4.  **成员处理**: 被选中的成员 Agent (如 `项目管理Agent`) 使用自己的模型和指令集处理这个特定的子任务。
5.  **结果返回**: 成员 Agent 的处理结果返回给 Leader Agent。
6.  **最终响应**: Leader Agent 整合（或直接采用）成员 Agent 的结果，生成最终的响应内容返回给用户。

**影响路由准确性的因素**: 
*   **Leader Agent 的理解能力**: Leader Agent 对用户意图和成员角色的理解准确度。
*   **成员 Agent 的 `role` 描述**: `role` 描述需要清晰、准确、具有区分度，以便 Leader Agent 能做出正确选择。
*   **成员 Agent 的 `instructions`**: 虽然主要用于指导成员自身行为，但也可能间接影响 Leader 对其能力的判断。
*   **Team 的 `instructions`**: 指导 Leader Agent 如何进行路由决策（例如，本项目的指令要求它直接为创建项目请求生成完整命令序列，并路由给项目管理 Agent）。

### Agent 名称识别挑战与解决方案

在 `api_server.py` 中，我们需要获取最终处理请求的专家 Agent 的名称，以便在前端显示。

*   **遇到的挑战**: 当使用 `lemonhall_assistant.run(stream=False)` 时，返回的 `TeamResponse` 对象虽然包含了执行细节，但直接从中获取处理 Agent 名称比较困难。
    *   尝试过访问 `response.member_name` 或 `response.agent_name`，但这些属性不存在或为空。
    *   尝试过访问 `response.member_responses[0].agent_id`，虽然能获取到一个 ID，但这个 ID 与初始化 Agent 时获取的 `.id` 不一致，导致基于初始 ID 创建的映射失效。
*   **最终解决方案**: 通过检查 `response.messages` 列表（记录了完整的交互过程）来解决。
    1.  遍历 `response.messages`。
    2.  查找 `role='tool'` 且 `tool_name='forward_task_to_member'` 的消息。
    3.  从该消息的 `tool_args['member_id']` 属性中提取值。**实验发现，这个 `member_id` 字段实际存储的是目标 Agent 的 *名称* (例如 "项目管理agent")**，而不是之前猜测的 ID。
    4.  将这个提取到的名称用于日志记录和 API 响应中的 `handling_agent` 字段。
    *   这种方法依赖于观察到的 `agno` 内部行为（`forward_task_to_member` 工具调用的参数结构），如果 `agno` 版本更新导致行为变化，可能需要调整。

## 文件结构

*   `src/`: 包含主要的 Python 源代码。
    *   `__init__.py`: 标记 `src` 为 Python 包。
    *   `teams_consoles.py`: 命令行交互版本的主程序，定义了 Agent 团队和控制台交互逻辑。
    *   `api_server.py`: FastAPI 服务器，提供 Web API 并托管前端。
    *   `volcengine_provider.py`: 封装了火山引擎 DeepSeek 模型初始化的类。
    *   `lemonhall_memory.db`: SQLite 数据库文件，用于存储记忆 (会自动创建)。
*   `frontend/`: 包含前端静态文件。
    *   `index.html`: ChatUI 前端页面。
    *   `assets/`: 存放静态资源。
        *   `avatars/`: 存放 SVG 头像文件。
            *   `user.svg`
            *   `assistant.svg`
*   `.venv/`: Python 虚拟环境目录 (由 `uv venv` 创建)。
*   `README.md`: 本说明文件。
*   `requirements.txt`: (需要自行创建或生成) Python 依赖列表。

--- 
*请将 `<your-repository-url>` 替换为你的实际仓库地址。*
