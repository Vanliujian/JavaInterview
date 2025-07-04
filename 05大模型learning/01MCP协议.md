# 初识模型

1. 什么是大语言模型（LLM）

   基于大量数据进行预训练的深度学习模型。

2. 应用有哪些？

   - 文案写作
   - `知识库回答`：可以根据数字存档中的信息帮助回答特定问题
   - 文本分类
   - 代码生成
   - 文本生成

3. 如何训练模型？

   学习模型：

   - 零样本学习：Base LLM 无需明确训练即可响应各种请求，通常是通过提示，但是答案的准确性各不相同。
   - 少量样本学习：通过提供一些相关的训练示例，基础模型在该特定领域的表现显著提升。
   - `微调`：少量样本学习的扩展，其中数据科学家训练基础模型，使模型使用与特定应用相关的其他数据来调整其参数。

## MCP协议（Multi-Call protocol）

### 基础概念

让大模型更好的使用各类工具

核心：告诉LLM有哪些接口可以被调用，用结构化方式描述他们，让模型自己决定是否需要调用，需要传哪些参数。

举例：

​	给模型两个tool，一个是天气查询，一个是食谱查询。当模型遇到类似提问，他会自己判断调用哪个接口获取。

### MCP Tool调用流程

![image-20250421234606344](图片/image-20250421234606344.png)

mcp市场有很多

- https://mcp.so/
- https://mcpmarket.com/

正常mcp都是用python或者node来写，所以对应启动方式有uvx和npx。

安装工具：https://github.com/astral-sh/uv

```shell
liujiandeMacBook-Pro:~ liujian$ curl -LsSf https://astral.sh/uv/install.sh | sh
downloading uv 0.6.14 x86_64-apple-darwin
no checksums to verify
installing to /Users/liujian/.local/bin
  uv
  uvx
everything's installed!

To add $HOME/.local/bin to your PATH, either restart your shell or run:

    source $HOME/.local/bin/env (sh, bash, zsh)
    source $HOME/.local/bin/env.fish (fish)
liujiandeMacBook-Pro:~ liujian$ source $HOME/.local/bin/env
liujiandeMacBook-Pro:~ liujian$ uv --version
uv 0.6.14 (a4cec56dc 2025-04-09)
liujiandeMacBook-Pro:~ liujian$ 
```

使用：

```shell
liujiandeMacBook-Pro:~ liujian$ uvx pycowsay 'hello world!'
Installed 1 package in 8ms
/Users/liujian/.cache/uv/archive-v0/jPSMWE8wFzsf8GjxF6Suy/lib/python3.13/site-packages/pycowsay/main.py:23: SyntaxWarning: invalid escape sequence '\ '
  """

  ------------
< hello world! >
  ------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||

liujiandeMacBook-Pro:~ liujian$ 
```

### MCP协议是和模型无关的

1. 函数的注册：规定了每个mcp server有哪些函数可以使用
2. 函数的使用：如何调用这些函数

`mcp协议并没有规定如何与模型进行交互`
