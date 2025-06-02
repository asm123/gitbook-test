# Model Context Protocol (MCP)

Developed by Anthropic: [Nov 2024](https://www.anthropic.com/news/model-context-protocol)[Specifications](https://modelcontextprotocol.io/introduction)

***

#### What is it

Think of MCP like a USB-C port for AI applications. Just as USB-C provides a standardized way to connect your devices to various peripherals and accessories, MCP provides a standardized way to connect AI models to different data sources and tools.

* MCP defines standard schema for querying information from MCP servers.
* MCP servers may be maintained by services that expose information from their tools.
* Using MCP in LLMs:
  * LLM lists all the tools in the beginning.
  * LLM sets system context in order to query information from MCP servers in response to user prompt.
* Every MCP server has to expose three capabilities:
  * Tool - functions that LLMs can call to perform specific actions.
  * Resource - data sources that LLMs can access.
  * Prompt - pre-defined templates for AI interactions

#### Where is it useful

* For questions which require latest data, e.g., current stock price, travel recommendations considering latest conditions.
* As a standardized integration layer for agents. Complements agent orchestration tools. It is not an agent framework on its own.

#### References

[What Is MCP, and Why Is Everyone – Suddenly!– Talking About It?](https://huggingface.co/blog/Kseniase/mcp)

[#Model Context Protocol Clearly Explained | MCP Beyond the Hype](https://www.youtube.com/watch?v=tzrwxLNHtRY)
