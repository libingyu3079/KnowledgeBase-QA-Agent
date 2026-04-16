# 产品周期智能体 (Product Lifecycle Agent)

这是一个基于大模型（LLM）的动态知识库系统。本项目的设计灵感来源于 AI 大神 Andrej Karpathy 提出的 **「LLM Wiki」** 理念。

> **核心思想**：真正的个人知识库不是简单地“存资料”或“RAG检索”，而是把 LLM 当作“知识工程师”，让它持续替你**编译、维护、交叉链接、回写和扩展**一整个知识空间。

参考文章：[【AI技能】传疯了！AI 大神 Karpathy ：真正的知识库不是存资料，而是把知识变成一套会自己生长的系统](https://mp.weixin.qq.com/s/ZQxfbggPo8ZxEaa8ZqKIYw)

---

## 🌟 核心架构设计 (五层模型)

本项目摒弃了传统的“丢文件+检索”模式，实现了真正能够用起来的智能知识库五层架构：

1. **原始材料层 (Raw Layer)**：`data/raw/` 目录。支持上传文档（TXT / PDF / DOCX），系统会自动解析为纯文本材料。
2. **编译层 (Compiled Layer)**：`data/compiled/` 目录。LLM 读取原始资料后，自动增量“编译”出结构化数据（包含摘要、核心实体、多维索引关键词），存为 JSON 格式。
3. **路由层 (Routing Layer)**：`data/routing/INDEX.md`。通过全局路由文件定义智能体的行为准则、角色定义和兜底策略。
4. **输出层 (Output Layer)**：支持多轮对话、流式输出的智能问答接口。具备动态意图澄清能力，能够引导用户细化提问。
5. **回写层 (Write-back Layer)**：持续扩展的知识沉淀（可通过系统扩展实现新发现和验证结果的回写归档）。

## ✨ 核心特性

- **双引擎驱动**：
  - **文件流驱动 (`main.py`)**：提供从前端上传文档、解析、LLM自动编译提取实体、到问答检索的全链路演示。
  - **数据库驱动 (`main_db.py`)**：通过 SQLite 提供更高效的持久化知识检索方案，适合生产环境扩展。
- **智能编译与检索**：不依赖传统的向量数据库和复杂的 RAG 切片，而是让 LLM 提取文档的核心主题词和实体，通过**双向关键词匹配**机制实现高准度召回。
- **多格式文档解析**：内置 PDF 和 DOCX 格式解析器，自动提取文本并保留表格结构。
- **多轮上下文理解**：前端流式对话，大模型根据历史记录中的代词（如“它”、“这个”）精准还原用户意图。

## 🛠️ 技术栈

- **后端**：Python 3, FastAPI, Uvicorn
- **数据库**：SQLite3 (`init_db.py` / `main_db.py`)
- **大模型接入**：OpenAI SDK 兼容接口（当前配置为对接阿里云百炼 DashScope 的 `qwen3-vl-plus` 模型）
- **文档解析**：`PyPDF2`, `python-docx`
- **前端**：原生 HTML + JS (`static/` 目录)

## 📂 项目结构

```text
.
├── data/
│   ├── raw/                 # 1. 原始材料层 (存放txt/pdf/docx解析后的纯文本)
│   ├── compiled/            # 2. 编译层 (存放LLM提取的摘要、实体和索引结构JSON)
│   └── routing/             # 3. 路由层 (存放 INDEX.md 智能体指令规则)
├── static/                  # 静态资源层 (前端页面)
│   ├── index.html           # 文件流版前端 UI
│   └── index_db.html        # 数据库版前端 UI
├── 知识库文档/              # 提供的示例文档 (5G新通话、5G消息等)
├── main.py                  # 核心服务入口 (五层架构全链路版本)
├── main_db.py               # 核心服务入口 (数据库驱动的高效检索版本)
├── init_db.py               # 数据库初始化脚本 (将编译层数据导入SQLite)
├── product_knowledge.db     # SQLite 知识库数据库文件 (自动生成)
├── requirements.txt         # Python 依赖清单
└── README.md                # 项目说明文档
```

## 🚀 快速开始

### 1. 安装依赖

请确保您的环境中已安装 Python 3.8+。在项目根目录下执行：

```bash
pip install -r requirements.txt
```
*(如果缺少 `PyPDF2` 或 `python-docx`，可手动执行 `pip install fastapi uvicorn openai PyPDF2 python-docx nest_asyncio`)*

### 2. 配置大模型 API Key

项目默认使用阿里云百炼平台的兼容接口。请在 `main.py` 和 `main_db.py` 中找到以下代码，并替换为您自己的 API Key：

```python
ALIYUN_API_KEY = "您的阿里云DashScope API KEY"
```

### 3. 运行方式一：全链路文件驱动版 (推荐体验完整编译流程)

该版本支持通过前端页面直接上传文件、点击编译并进行对话。

```bash
python main.py
```
启动后，在浏览器访问：[http://localhost:8081](http://localhost:8081)

### 4. 运行方式二：数据库高效驱动版

如果您已经有了编译好的数据，可以通过 SQLite 数据库提供更高效的检索。

**第一步：初始化数据库**
```bash
python init_db.py
```
*(这会读取 `data/compiled/` 下的 JSON 文件，并生成 `product_knowledge.db`)*

**第二步：启动数据库版服务**
```bash
python main_db.py
```
启动后，在浏览器访问：[http://localhost:8082](http://localhost:8082)

## 💡 使用小贴士

- **测试数据**：您可以使用任意与产品文档相关的docx文件或者pdf文件进行测试，也可以根据需求修改data文件夹中routing中的提示词来适配您的智能体。
- **意图澄清**：在对话框中仅输入“5G消息”，体验智能体根据文档自动提取多维度主题并向您反问的“动态意图澄清”能力。

## 🤝 致谢

本项目核心思想受 Andrej Karpathy 的启示，探索个人知识管理（PKM）与 LLM 深度融合的下一代范式。感谢开源社区提供的优秀工具！
