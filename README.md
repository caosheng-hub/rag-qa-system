# 基于 RAG 的智能客服与知识库系统

> 一个从零搭建的企业级 RAG（检索增强生成）应用，支持文档知识库构建与多轮对话问答，具备去重、流式响应、对话记忆等特性。

## 1. 项目背景与价值

在传统客服场景中，基于关键词匹配的 FAQ 系统无法理解复杂问题，且维护成本高；直接使用大模型则存在知识更新不及时、容易产生幻觉的问题。本项目通过 **RAG 架构**，将外部知识库与大模型结合，实现：
- 知识实时注入：上传文档即可让大模型学习新知识
- 可溯源的回答：答案基于检索到的具体文档片段
- 低成本迭代：无需微调模型，更新知识库即可

适用于企业产品说明、内部知识库问答、电商客服等场景。

## 2. 系统架构与技术栈

### 整体架构图

[用户上传文档] → 文本分割 → 向量化 → Chroma 存储
↓
[用户提问] → 向量检索 → 上下文增强 → LLM 生成 → 流式返回
↑
对话历史管理（本地持久化）

### 技术选型及理由

| 组件 | 技术 | 选型理由 |
|------|------|----------|
| 前端界面 | Streamlit | 快速构建交互式 Web 应用，无需前端知识，支持实时流式输出 |
| 编排框架 | LangChain | 提供成熟的 RAG 链、会话管理、输出解析等抽象，降低开发复杂度 |
| 向量数据库 | Chroma（本地） | 轻量级、嵌入式，无需额外部署，适合项目演示与小型生产环境 |
| 嵌入模型 | DashScope text-embedding-v4 | 阿里云提供，中文语义理解强，API 稳定，成本可控 |
| 对话模型 | DashScope qwen3-max | 通义千问最新版本，推理能力强，支持流式生成 |
| 历史存储 | 自定义 JSON 文件 | 无额外依赖，实现 `BaseChatMessageHistory` 接口，便于扩展 |

### 项目模块划分

- `knowledge_base.py`：知识库核心逻辑（文本分块、MD5 去重、向量化存储）
- `vector_stores.py`：封装 Chroma 检索器，支持配置召回数量
- `rag.py`：构建完整 RAG 链，集成提示模板、对话历史、流式输出
- `file_history_store.py`：自定义聊天历史持久化类
- `config_data.py`：集中管理配置参数（分块大小、模型名称、路径等）
- 两个 Streamlit 应用：上传服务与问答服务，解耦知识注入与问答交互

## 3. 核心功能与实现亮点

### 🔹 智能知识库构建
- **递归字符分割**：优先按段落、句子分割，保留语义完整性；支持重叠窗口（overlap=100），避免信息断裂。
- **MD5 内容去重**：计算全文 MD5 并持久化，重复上传自动跳过，避免向量库膨胀。
- **元数据标记**：为每个文本块记录来源文件名、上传时间、操作人，便于追溯。

### 🔹 检索增强生成
- **多路召回**：默认返回 top-2 相似文档，可通过参数调整。
- **动态提示模板**：同时注入检索到的 `context` 和对话 `history`，引导模型基于事实回答。
- **流式输出**：利用 LangChain `stream` API + Streamlit `write_stream`，实现打字机效果，提升用户体验。

### 🔹 对话记忆管理
- **自定义历史类**：继承 `BaseChatMessageHistory`，将消息序列化为 JSON 存储到本地文件。
- **会话隔离**：通过 `session_id` 区分不同用户（当前为单用户演示，设计上支持多会话）。
- **LangChain 集成**：使用 `RunnableWithMessageHistory` 自动注入历史，无需手动管理。

## 4. 个人贡献与技术难点

> 本项目为个人独立完成，以下为关键贡献与解决问题的思路。

### 难点 1：如何避免重复向量化相同内容？
**解决方案**：在存入向量库前，计算整个文本的 MD5，维护一个 `md5.text` 文件记录已处理的 MD5。每次上传先检查，存在则直接返回，否则继续流程。  
**效果**：避免存储冗余，节约 API 调用成本。

### 难点 2：流式输出时如何同时保存完整回答？
**解决方案**：编写 `capture` 生成器函数，在 `yield` 每个 chunk 的同时追加到 `cache_list`，最后用 `''.join(cache_list)` 获取完整内容存入会话状态。  
**效果**：前端实时显示，后台完整记录，两全其美。

### 难点 3：LangChain 中如何注入对话历史？
**解决方案**：首先实现 `FileChatMessageHistory` 类，提供 `add_messages` 和 `messages` 属性；然后在 `RunnableWithMessageHistory` 中传入 `get_history` 函数，并指定 `input_messages_key` 和 `history_messages_key`。  
**效果**：对话历史自动保存与加载，无需侵入业务代码。

### 难点 4：提示模板中如何同时处理 context 和 history？
**解决方案**：使用 `RunnableLambda` 对链中流转的数据结构进行转换。检索阶段从用户输入中提取问题；生成阶段将 `{input, context, history}` 重组为 prompt 所需的格式。  
**效果**：链式调用清晰，各环节职责单一。

## 5. 运行与测试

### 环境要求
- Python 3.9+
- 阿里云 DashScope API Key（[申请地址](https://dashscope.console.aliyun.com/)）

### 安装步骤
```bash
git clone https://github.com/yourname/rag-qa-system.git
cd rag-qa-system
pip install -r requirements.txt
export DASHSCOPE_API_KEY="sk-xxx"
