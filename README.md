# 基于 RAG 的智能客服与知识库管理系统

本项目是一个完整的 **RAG（检索增强生成）** 应用，基于 Streamlit + LangChain + DashScope + Chroma 构建。  
支持上传 `.txt` 文档构建知识库，并提供多轮对话式问答，具备聊天历史持久化、文档去重、流式输出等特性。

## 功能特点

- 📄 **知识库上传**：上传 TXT 文件，自动分块、向量化，存储到 Chroma 向量数据库。
- 🔍 **智能问答**：基于用户问题检索相关文档片段，结合对话历史，调用通义千问 Qwen3-Max 生成回答。
- 💬 **多轮对话**：使用本地文件持久化对话历史，支持上下文记忆。
- ⚡ **流式响应**：前端实时逐字输出大模型回答。
- 🧠 **MD5 去重**：避免重复上传相同内容，节省存储与计算资源。

## 技术栈

- 前端：Streamlit
- 框架：LangChain
- 向量数据库：Chroma（本地持久化）
- 嵌入模型：DashScope `text-embedding-v4`
- 对话模型：DashScope `qwen3-max`
- 历史管理：自定义 `FileChatMessageHistory`

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/caosheng-hub/rag-qa-system.git
cd rag-qa-system
