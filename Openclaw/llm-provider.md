# OpenClaw — LLM Providers

## What is an LLM Provider?
OpenClaw is just the agent framework — it needs an AI brain to work.
You plug in one or more LLM providers to give it intelligence.

## How the model format works
Always written as: provider/model
Examples:
- anthropic/claude-sonnet-4-6
- openai/gpt-4o
- ollama/llama3

## Providers I'm exploring
| Provider   | Cost        | Best For               | API Key Needed? |
|------------|-------------|------------------------|-----------------|
| Anthropic  | Paid        | Best overall quality   | Yes             |
| OpenAI     | Paid        | Widely supported       | Yes             |
| Ollama     | Free        | Privacy, local use     | No              |
| OpenRouter | Pay per use | Access many models     | Yes             |
| Qwen       
| Kimi
| 

## My current setup
- Primary: Anthropic (Claude)
- Plan to add: Ollama for local/private tasks

## How to switch models via CLI
openclaw models set anthropic/claude-sonnet-4-6

## How to see all available models
openclaw models list

## Learned on
April 2026