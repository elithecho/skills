---
name: ruby-llm
description: "Ruby gem providing a unified API for interacting with multiple AI providers (OpenAI, Anthropic, Google Gemini, Ollama, and more). Use when working with Ruby applications that need chat conversations, embeddings, image generation, audio transcription, content moderation, tool/function calling, Rails integration, or async operations."
---

# RubyLLM

## Quick Start

```ruby
# Simple chat
chat = RubyLLM.chat
response = chat.ask "What is Ruby?"
puts response.content

# With a specific model
chat = RubyLLM.chat(model: 'gpt-5')
response = chat.ask "Explain Ruby metaprogramming"

# Generate images
image = RubyLLM.paint("A sunset over mountains")

# Create embeddings
embedding = RubyLLM.embed("Ruby is elegant and expressive")
```

## Installation

```ruby
bundle add ruby_llm
```

## Configuration

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = ENV['OPENAI_API_KEY']
  config.anthropic_api_key = ENV['ANTHROPIC_API_KEY']
end
```

## Core Features

- **Chat**: Conversational AI interactions with message history
- **Tools**: Let AI models call your Ruby code
- **Streaming**: Real-time response streaming
- **Embeddings**: Text vectorization for similarity search
- **Image Generation**: AI image creation (DALL-E, etc.)
- **Audio Transcription**: Speech to text
- **Moderation**: Content safety checking
- **Extended Thinking**: Reasoning models

## Supported Providers

OpenAI, Anthropic, Google Gemini, AWS Bedrock, DeepSeek, Mistral, Ollama (local), OpenRouter, Perplexity, xAI (Grok), Azure AI, GPUStack

## Reference Documentation

For detailed information on each feature, see the reference files:

- **Getting Started**: @rules/getting-started.md
- **Configuration**: @rules/configuration.md
- **Chat**: @rules/chat.md
- **Tools**: @rules/tools.md
- **Streaming**: @rules/streaming.md
- **Embeddings**: @rules/embeddings.md
- **Image Generation**: @rules/image-generation.md
- **Audio Transcription**: @rules/audio-transcription.md
- **Moderation**: @rules/moderation.md
- **Extended Thinking**: @rules/thinking.md
- **Rails Integration**: @rules/rails.md
- **Async**: @rules/async.md
- **Error Handling**: @rules/error-handling.md
- **Model Registry**: @rules/models.md
- **Agentic Workflows**: @rules/agentic-workflows.md
