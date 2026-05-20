# RubyLLM Documentation

RubyLLM provides a unified Ruby API for interacting with multiple AI providers (OpenAI, Anthropic, Google Gemini, and more). This documentation covers everything you need to build AI-powered Ruby applications.

## Overview

RubyLLM is a Ruby gem that abstracts away the differences between various LLM providers, giving you a single, consistent interface for:
- Chat conversations
- Embeddings generation
- Image generation
- Audio transcription
- Content moderation
- Tool/function calling

## Key Features

- **Provider Agnostic**: Same code works across OpenAI, Anthropic, Gemini, Ollama, and more
- **Minimal Dependencies**: Only Faraday, Zeitwerk, and Marcel
- **Rails Integration**: Built-in ActiveRecord support with `acts_as_chat`
- **Async Support**: Fiber-based concurrency for high-throughput applications
- **Tool Calling**: Let AI models call your Ruby code
- **Structured Output**: JSON schemas for guaranteed response formats

## Documentation Index

### Getting Started
- [Getting Started](getting-started.md) - Quick start guide
- [Overview](overview.md) - Architecture and design principles
- [Configuration](configuration.md) - API keys, defaults, and settings

### Core Features
- [Chat](chat.md) - Conversational AI interactions
- [Tools](tools.md) - Function calling and tool definitions
- [Streaming](streaming.md) - Real-time response streaming
- [Embeddings](embeddings.md) - Text vectorization
- [Image Generation](image-generation.md) - AI image creation
- [Audio Transcription](audio-transcription.md) - Speech to text
- [Moderation](moderation.md) - Content safety checking
- [Extended Thinking](thinking.md) - Reasoning models

### Advanced Topics
- [Rails Integration](rails.md) - ActiveRecord persistence
- [Async](async.md) - Concurrent operations
- [Error Handling](error-handling.md) - Graceful error management
- [Model Registry](models.md) - Model discovery and capabilities
- [Agentic Workflows](agentic-workflows.md) - Orchestrating multiple agents
- [Upgrading](upgrading.md) - Version migration guides

## Quick Example

```ruby
# Simple chat
chat = RubyLLM.chat
response = chat.ask "What is Ruby?"
puts response.content

# With a specific model
chat = RubyLLM.chat(model: 'claude-sonnet-4-5')
response = chat.ask "Explain Ruby metaprogramming"

# Generate images
image = RubyLLM.paint("A sunset over mountains")

# Create embeddings
embedding = RubyLLM.embed("Ruby is elegant and expressive")
```

## Installation

```ruby
# Gemfile
gem 'ruby_llm'
```

```bash
bundle install
```

## Configuration

```ruby
# config/initializers/ruby_llm.rb
RubyLLM.configure do |config|
  config.openai_api_key = ENV['OPENAI_API_KEY']
  config.anthropic_api_key = ENV['ANTHROPIC_API_KEY']
end
```

## Supported Providers

- OpenAI (GPT, DALL-E, Whisper)
- Anthropic (Claude)
- Google Gemini
- AWS Bedrock
- DeepSeek
- Mistral
- Ollama (local)
- OpenRouter
- Perplexity
- xAI (Grok)
- Azure AI
- GPUStack

## Related Documentation

- [Official RubyLLM Website](https://rubyllm.com/)
- [GitHub Repository](https://github.com/crmne/ruby_llm)
- [RubyLLM Blog](https://paolino.me)
