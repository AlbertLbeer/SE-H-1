# Code Review Agent

一个面向课程作业的代码审查 Agent。它使用 DeepSeek V4.1-Flash 分析 Python 代码，并通过本地工具读取文件、验证 Python 语法，最终生成结构化审查报告

## 主要功能

- 基本 Agent 循环：用户输入、模型推理、工具调用、结果反馈和最终输出
- `read_file`：安全读取工作区内的 UTF-8 文本文件
- `check_python_syntax`：使用 Python AST 检查语法，不执行代码
- 支持同一次程序运行期间的多轮对话历史
- LLM 请求失败自动重试
- 限制可读取目录和文件大小
- 提供 Mock 模式和自动化测试

## 项目结构

```text
code-review-agent/
├── app/
│   ├── tools/
│   │   ├── file_reader.py
│   │   └── python_syntax.py
│   ├── agent.py
│   ├── config.py
│   ├── llm_client.py
│   ├── main.py
│   ├── mock_llm.py
│   └── prompts.py
├── docs/
│   └── USAGE.md
├── examples/
├── review_files/
├── tests/
├── .env.example
├── .gitignore
├── Design.md
└── requirements.txt
```

## 快速开始

进入项目目录：

```powershell
cd D:\code-review-agent
```

把待审查文件放入 `review_files`，例如 `review_files/my_code.py`。该目录中的文件会被 Git 忽略

在当前 PowerShell 会话配置API， 以下为DeepSeek示例：

```powershell
$env:LLM_BASE_URL = "https://api.deepseek.com"
$env:LLM_MODEL = "deepseek-flash"
$env:LLM_API_KEY = "替换为你的API Key"
```

运行审查：

```powershell
python -m app.main `
  --file review_files/my_code.py `
  --request "请审查这个 Python 文件，重点检查语法、潜在 Bug、安全问题和可维护性。"
```

## 多轮记忆与交互模式

单次使用 `--file` 参数时，程序生成审查报告后会直接退出。要继续追问上一轮的审查结果，需要启动交互模式：

```powershell
python -m app.main
```

在同一个交互会话中连续输入，例如：

```text
你：请审查 review_files/my_code.py
Agent：……

你：请详细解释刚才报告中的第一个问题
Agent：……

你：请给出这个问题的修改建议
Agent：……
```

输入 `quit`、`exit` 或 `q` 可以退出交互模式

历史消息保存在程序运行时的内存中，只对当前交互会话有效。退出程序后历史消息会清空，下次启动不会恢复

架构设计见 [Design.md](Design.md)

## 无 API Key 演示

```powershell
python -m app.main --mock --file examples/ex1.py
```

Mock 模式只验证 Agent 工具调用流程，不生成真实的代码审查结论

## 测试

```powershell
python -m unittest discover -s tests -v
```



## 当前限制

- 主要审查单个 Python 文件
- 不执行待审查代码，也不自动修改文件
- 历史消息只保存在内存中，退出程序后清空
- 审查意见由大语言模型生成，重要问题仍需人工确认
