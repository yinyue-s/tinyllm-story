# TinyLLM-Story · AI 儿童故事创作平台

基于 FastAPI + LangChain + FAISS 的儿童故事创作与聆听平台：浏览故事库、输入主题一键生成童话、自动配图、浏览器朗读，并与具备 RAG 检索与工具调用能力的智能助手对话。

## 功能

- **故事库**：内置 700 篇故事、7 大分类，支持分类筛选、关键词搜索与分页浏览
- **AI 故事创作**：输入主题、分类、主角、篇幅生成完整故事，支持 SSE 流式输出与打字机效果
- **三级降级生成链**：远程大模型 API、本地 HuggingFace 模型、本地模板引擎逐级兜底，没有 API Key 也能完整跑通
- **智能助手**：LangChain 工具调用（5 个工具）与手写 ReAct 循环，支持 RAG 语义检索与多轮对话记忆
- **配图与朗读**：远程图片 API 生成插图，未配置时用本地 SVG 插画兜底；故事详情页支持浏览器语音朗读
- **用户体系**：注册登录（PBKDF2-SHA256 加盐哈希 + JWT）、收藏、生成历史、个人中心

## 技术栈

| 层次 | 技术 |
| --- | --- |
| 后端 | Python 3.11、FastAPI、SQLAlchemy 2.0、Uvicorn |
| AI 能力 | LangChain Core、langchain-openai、FAISS、langchain-text-splitters、transformers（可选本地模型） |
| 数据库 | SQLite |
| 前端 | 原生 HTML / CSS / JavaScript，无框架无构建工具 |

## 目录结构

```
backend/
  main.py            # FastAPI 应用装配：中间件、路由注册、静态文件托管
  config.py          # 环境变量与常量
  database.py        # 引擎、会话工厂、数据初始化
  models.py          # SQLAlchemy 模型（用户、故事、配图、收藏、历史、会话、消息）
  auth_util.py       # 密码哈希与 JWT 签发校验
  routers/           # 路由层：system / auth / stories / generate / assistant
  services/          # 业务层：文本生成、配图、模型工厂、RAG、Agent
fronted/             # 前端页面（首页、创作、详情、助手、登录、注册、个人中心）
Langchain/           # LangChain 学习示例：模型调用、提示词链、记忆、RAFT
src/data/            # 故事数据集
```

## 快速开始

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
uvicorn backend.main:app --reload
```

启动后访问 http://127.0.0.1:8000

`.env` 中的模型 API Key 可以留空：此时文本生成走本地模板引擎、向量检索走内置字符 n-gram 嵌入，全部核心功能离线可用。

## 配置项

完整配置见 `.env.example`，常用的几项：

| 变量 | 说明 |
| --- | --- |
| `TINYLLM_DATABASE_URL` | 数据库连接串，默认 `sqlite:///tinyllm.db` |
| `TINYLLM_JWT_SECRET` | JWT 签名密钥，部署前必须替换 |
| `TINYLLM_TEXT_API_URL` / `_KEY` / `_MODEL` | 兼容 OpenAI 协议的对话模型（DeepSeek、通义千问、智谱等） |
| `TINYLLM_IMAGE_API_URL` / `_KEY` | 配图接口，未配置时使用本地 SVG 兜底 |
| `TINYLLM_MODEL_PATH` | 本地 HuggingFace 模型路径，作为二级降级 |
| `TINYLLM_RAG_CHUNK_SIZE` / `_OVERLAP` / `_TOP_K` | RAG 切块与检索参数 |

## 主要接口

基址 `/api/v1`。

| 功能 | 方法 | 路径 |
| --- | --- | --- |
| 注册 / 登录 | POST | `/auth/register`、`/auth/login` |
| 个人中心 | GET | `/auth/profile` |
| 故事列表 / 详情 | GET | `/stories`、`/stories/{id}` |
| 收藏切换 / 收藏列表 / 浏览历史 | POST GET | `/stories/{id}/favorite`、`/stories/favorites`、`/stories/history` |
| 生成故事 / 流式生成 | POST GET | `/generate/story`、`/generate/story/stream`（SSE） |
| 生成配图 | POST | `/generate/images` |
| 助手会话与对话 | POST GET DELETE | `/assistant/sessions`、`/assistant/sessions/{id}/chat` |

## 说明

- 仓库不含虚拟环境、SQLite 数据库文件、运行时生成的 FAISS 索引，以及体积较大的原始语料（`src/data/gushi365_data/`，约 80MB）。
- 密钥类配置统一通过环境变量注入，`.env` 已在 `.gitignore` 中排除，提交前请勿写入真实 Key。
- 课程中的 ViT、DistilBERT、YOLO 等深度学习练习脚本与本项目无关，未纳入本仓库。
