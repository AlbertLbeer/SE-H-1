# 设计说明

## 1. 项目目标

本项目实现一个代码审查 Agent。用户可以提交代码片段或代码文件路径，Agent 通过大语言模型分析代码质量，指出潜在问题并给出修改建议

## 2. Agent 循环

```text
用户请求
  ↓
LLM 判断是否需要工具
  ↓
调用 read_file 读取代码
  ↓
调用 check_python_syntax 验证 Python 语法
  ↓
把工具结果加入对话上下文
  ↓
LLM 继续分析或返回最终报告
```

Agent 设置最大循环次数，避免模型反复调用工具。工具调用失败会把错误信息返回给模型，让模型决定如何继续；模型请求本身由 LLM 客户端负责有限次数重试

## 3. 模块职责

- `app/main.py`：命令行入口和交互循环
- `app/agent.py`：维护消息、调用模型、执行工具和限制循环次数
- `app/llm_client.py`：封装API 的 HTTP 请求
- `app/tools/file_reader.py`：读取工作区内的文本代码文件
- `app/tools/python_syntax.py`：使用 `ast.parse` 检查 Python 语法，不执行用户代码
- `app/prompts.py`：系统 Prompt 和审查输出要求
- `app/config.py`：从环境变量加载配置

## 4. 安全边界

只实现文件读取和语法解析，不执行用户代码。文件路径必须位于工作区目录内，文件大小也有上限后续如果增加代码执行工具，应放在隔离进程中，并设置超时、资源限制和允许的命令白名单

## 5. 后续扩展

- 增加静态检查工具，例如 Ruff 或 Pylint
- 增加 Markdown/JSON 格式报告
- 将命令行入口替换为简单 Web 界面
- 增加更多针对边界情况的测试
